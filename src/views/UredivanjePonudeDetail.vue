<template>
  <div class="container">
    <sidebar />
    <router-link :to="{ name: 'UredjivanjePonude' }"><Natrag /> </router-link>
    <div class="sredina">
      <div class="PozicijaNatrag"></div>
      <h1 class="naslovStranice">{{ Naslov }}</h1>
      <div class="top">
        <div class="promjena">
          <div
            class="Prvo ho"
            v-if="!VidljivBrisanje"
            @click="VidljivBrisanje = !VidljivBrisanje"
          >
            Brisanje kategorije
          </div>

          <div class="Drugo" v-if="VidljivBrisanje">
            Obriši trenutnu kategoriju

            <div class="red">
              <div v-if="VidljivBrisanje" @click="BrisanjeKategorije">
                <potvrdi />
              </div>
              <div
                v-if="VidljivBrisanje"
                @click="VidljivBrisanje = !VidljivBrisanje"
              >
                <odustani />
              </div>
            </div>
          </div>

          <div
            class="Prvo ho"
            v-if="!VidljivPromjena"
            @click="VidljivPromjena = !VidljivPromjena"
          >
            Promjena naziva
          </div>

          <div class="Drugo" v-if="VidljivPromjena">
            Promjena naziva
            <input
              class="UpisiNaslov"
              v-model="NaslovNovi"
              placeholder=" Ime potkategorije"
              type="text"
            />

            <div class="red">
              <div v-if="VidljivPromjena" @click="PromjenaNazivaPotkategorije">
                <potvrdi />
              </div>
              <div
                v-if="VidljivPromjena"
                @click="VidljivPromjena = !VidljivPromjena"
              >
                <odustani />
              </div>
            </div>
          </div>

          <div class="SlikaDodaj">
            <div class="ButtoniSlike">
              <label for="files" v-if="!Vidljiv" class="ButtonSlike1"
                >Odabir slike</label
              >
              <input
                id="files"
                class="ButtonSlike"
                style="visibility: hidden"
                type="file"
                accept="image/*"
                title="!"
                @change="postaviSliku"
              />
            </div>

            <div class="t" v-if="Vidljiv">Potvrdite dodavanje slike</div>
            <div class="ButtoniSlika">
              <button class="ButtonSlike2" v-if="Vidljiv" @click="uploadImage">
                Potvrdi
              </button>
              <button
                class="ButtonSlike2"
                v-if="Vidljiv"
                @click="Vidljiv = !Vidljiv"
              >
                Odustani
              </button>
            </div>
          </div>
        </div>

        <div class="velicina" v-if="categoryImage">
          <img class="slikaKategorija" :src="categoryImage" alt="" />
          <i class="fas fa-times-circle SlikaX" @click="BrisnjeSlike"></i>
        </div>
      </div>
      <div class="Podnaslov">Kategorije</div>
      <div class="raspored">
        <DodavanjePotkategorije />
        <Potkategorija
          v-for="K in KarticePotkategorija"
          :key="K.id"
          :id="K.id"
          :info="K"
        />
      </div>
      <div class="Podnaslov">Uredi ponudu</div>
      <div class="UrediPonudu">
        <DodavanjeStavke />
        <Stavka
          v-for="Stavka in KarticeStavki"
          :key="Stavka.id"
          :idStavka="Stavka.id"
          :Stavke="Stavka"
        />
      </div>
    </div>
  </div>
</template>
<script>
import DodavanjeStavke from "@/components/DodavanjeStavke";
import Stavka from "@/components/Stavka";
import Potkategorija from "@/components/Potkategorija";
import Sidebar from "@/components/Sidebar";
import Natrag from "@/components/Natrag";
import {
  ref,
  storage,
  uploadBytes,
  query,
  doc,
  onSnapshot,
  collection,
  db,
  getDoc,
  getDownloadURL,
  updateDoc,
  deleteDoc,
  deleteObject,
} from "@/firebase";
import DodavanjePotkategorije from "@/components/DodavanjePotkategorije";
import potvrdi from "@/components/potvrdi";
import odustani from "@/components/odustani";
import { v4 as uuid } from "uuid";

export default {
  data() {
    return {
      Slika: null,
      categoryImage: null,
      KarticePotkategorija: [],
      Naslov: "",
      KarticeStavki: [],
      Vidljiv: false,
      VidljivBrisanje: false,
      VidljivPromjena: false,
      NaslovNovi: "",
    };
  },

  components: {
    Sidebar,
    DodavanjePotkategorije,
    Stavka,
    DodavanjeStavke,
    Potkategorija,
    potvrdi,
    odustani,
    Natrag,
  },
  mounted() {
    //DAJE TOČNO TRENUTAK KAD DA SE DATOTRKA PRIKAŽE NA ERKRANU
    this.PrikazPotkategorija(),
      this.PrikazNaslova(),
      this.PrikazStavke(),
      this.getSlika();
  },
  methods: {
    async PromjenaNazivaPotkategorije() {
      const ID = this.$route.params.id;
      const PozkategorijaDocRef = doc(collection(db, "Kategorija"), ID);

      try {
        await updateDoc(PozkategorijaDocRef, {
          Ime: this.NaslovNovi,
        });
        this.PrikazNaslova();
        this.VidljivPromjena = !this.VidljivPromjena;
      } catch (error) {
        console.log("GREŠKA PROMJENE POTKATEGORIJE");
      }
    },

    async BrisanjeKategorije() {
      const ID = this.$route.params.id;
      try {
        await deleteDoc(doc(collection(db, "Kategorija"), ID));
        this.$router.push({ name: "UredjivanjePonude" });
        alert("Obrisana je kategorija");
      } catch (error) {
        console.log("GREŠKA BRISANJA KATEGORIJE");
      }
    },
    async BrisnjeSlike() {
      const ID = this.$route.params.id;
      // Create a reference to the file to delete
      const desertRef = ref(storage, this.categoryImage);
      try {
        await deleteObject(desertRef);
        this.categoryImage = null;
        const KategorijaDocRef = doc(db, "Kategorija", ID);
        await updateDoc(KategorijaDocRef, { imageURL: "" });
      } catch (error) {
        console.log("GREŠKA BRISANJA SLIKE");
      }
    },

    postaviSliku(e) {
      this.Vidljiv = !this.Vidljiv;
      if (e.target.files[0] != null) {
        this.Slika = e.target.files[0];
      }
    },
    async getSlika() {
      const ID = this.$route.params.id;

      const categorySnap = await getDoc(doc(db, "Kategorija", ID));

      if (categorySnap.data().imageURL) {
        this.categoryImage = categorySnap.data().imageURL;
      }
    },
    async PrikazPotkategorija() {
      const ID = this.$route.params.id;
      const q = query(
        collection(doc(collection(db, "Kategorija"), ID), "Potkategorija")
      );
      onSnapshot(q, (querySnapshot) => {
        const PotkategorijaIme = [];

        querySnapshot.forEach((doc) => {
          PotkategorijaIme.push({ id: doc.id, ...doc.data() });
        });
        this.KarticePotkategorija = PotkategorijaIme;
      });
    },

    async PrikazStavke() {
      const ID = this.$route.params.id;
      const q = query(
        collection(doc(collection(db, "Kategorija"), ID), "Stavka")
      );
      onSnapshot(q, (querySnapshot) => {
        const Stavke = [];
        querySnapshot.forEach((doc) => {
          Stavke.push({ id: doc.id, ...doc.data() });
        });
        this.KarticeStavki = Stavke;
      });
    },

    async PrikazNaslova() {
      const docRef = doc(db, "Kategorija", this.$route.params.id);
      const docSnap = await getDoc(docRef);

      if (docSnap.exists()) {
        this.Naslov = docSnap.data().Ime;
        this.NaslovNovi = this.Naslov;
      } else {
        // doc.data() will be undefined in this case
        console.log("Nema rezultata!");
      }
    },

    async uploadImage() {
      this.Vidljiv = !this.Vidljiv;
      const ID = this.$route.params.id;
      if (!this.Slika) return;

      const imageName = uuid();
      const storageRef = ref(storage, imageName);

      await uploadBytes(storageRef, this.Slika);
      const downloadURL = await getDownloadURL(storageRef);

      updateDoc(doc(db, "Kategorija", ID), {
        imageURL: downloadURL,
      });
      this.getSlika();
    },
  },
};
</script>

<style scoped lang="scss">
* {
  margin: 0;
  padding: 0;
}

.container {
  display: flex;
}

.sredina {
  display: flex;
  flex-direction: column;
  align-items: center;

  margin: 3rem auto;
  max-width: 140vh;
  width: 120vh;

  padding-left: 25%;
}

.raspored {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  margin-top: 1rem;
  margin-left: 3rem;
  margin-right: 3rem;
  gap: 2rem;
}

.naslovStranice {
  font-family: "Amatic SC", cursive;
  font-size: 60px;
  color: #731642;
  font-weight: bold;
  margin-bottom: 1.5rem;
  margin-top: 3rem;
}

.UrediPonudu {
  font-size: 20px;
  font-weight: bold;
  color: #731642;
}

.slikaKategorija {
  width: 100%;
  height: 100%;
  margin-right: 1rem;
  margin-left: 1rem;
  border: 4px solid #731642;
  border-radius: 7px;
  object-fit: contain;
}

.ButtonSlike2 {
  background-color: #731642;
  color: #ffffff;
  font-size: 15px;
  border-radius: 7px;
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  border: none;
  font-weight: bold;
  outline: none;
  min-width: 5rem;
  max-width: 5rem;
}

.ButtonSlike1 {
  background-color: #ffffff;
  color: #731642;
  border: 3px solid #731642;
  border-radius: 10px;
  font-size: 17px;
  font-weight: bold;
  outline: none;
  cursor: pointer;
  padding-left: 2.3rem;
  padding-right: 2.3rem;
  padding-top: 0.9rem;
  padding-bottom: 0.9rem;
}
.ButtonSlike1:hover {
  background-color: #7217411f;
}
.ButtonSlike {
  width: 0;
  margin-top: 1rem;
}

.Prvo {
   display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: #731642;
  color: #ffffff;
  border-radius: 7px;
  padding: 1rem;
  border: none;
  max-width: 9rem;
  min-width: 9rem;
  outline: none;
  cursor: pointer;
}

.Drugo  {
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: #731642;
  color: #ffffff;
  border-radius: 7px;
  max-width: 13rem;
  min-width: 13rem;
  border: none;
  font-size: 15px;
  padding: 1rem;
  gap:0.5rem;
  outline: none;
  cursor: pointer;
  padding-bottom: 1.2rem;
  padding-top: 1.2rem;
}
.ho:hover {
  background-color: #721741d5;
}

.promjena {
  display: flex;
  flex-direction: column;
  font-weight: bold;
  align-items: center;
  gap: 1.1rem;
}

.top {
  display: flex;
  flex-direction: row;
  gap: 1rem;
  justify-content: center;
  margin-right: 1rem;
}

.t {
  margin-top: -2rem;
  margin-bottom: 0.5rem;
}

.SlikaX {
   color: #731642;
  position: absolute;
  right: -12px;
  top: 10px;
  z-index: 999;
  font-size: 18px;
  border-radius: 100%;
}

.Podnaslov {
  color: #731642;
  margin-top: 2rem;
  font-weight: bold;
  font-size: 20px;
}

.UpisiNaslov {
  border-radius: 7px;
  font-size: 13px;
  border: 1.5px solid #731642;
  padding: 0.3rem;
  width:92%;
}

.ButtoniSlika {
  display: flex;
  gap: 0.3rem;
  justify-content: center;
}

i {
  cursor: pointer;
}
.ButtonSlike2:hover {
  cursor: pointer;
  background-color: #aa6b88f3;
  color: #ffffff;
}



.red {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
}

.velicina{
  position: relative;
  width: 11rem;
  height: 11rem;
}
</style>
