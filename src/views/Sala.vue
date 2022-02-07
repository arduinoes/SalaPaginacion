<template>
  <Navbar />
  <div class="row justify-content-center">
    <BaseTarjeta
      class="col-4 m-2"
      v-for="obra in obras"
      :key="obra.id"
      :obra="obra"
    />
  </div>
  <div class="text-center">
    <v-pagination
      v-model="page"
      :length="paginas"
      @click="siguiente"
    ></v-pagination>
  </div>
</template>

<script>
import Navbar from "../components/Navbar.vue";
import BaseTarjeta from "../components/BaseTarjeta.vue";
import Pagination from "../components/Pagination.vue";
import {
  collection,
  getDocs,
  query,
  orderBy,
  limit,
  startAt,
  startAfter,
} from "firebase/firestore";
import { db } from "../main";
export default {
  name: "Sala",
  components: {
    Navbar,
    BaseTarjeta,
    Pagination,
  },
  data() {
    return {
      total: 0,
      paginas: 0,
      limit: 3,
      page: 1,
      lastVisible: "",
      firstVisible: "",
      obras: [],
      obra: {
        title: "",
        author: "",
        date: "",
        photo: "",
      },
    };
  },
  methods: {
    async obtenerDatos() {
      const q = await query(collection(db, "obras"), orderBy("author"));

      const documentSnapshots = await getDocs(q);
      const totalDocumentos = documentSnapshots.docs.length;
      this.total = totalDocumentos;
      this.paginas = Math.ceil(this.total / this.limit);
      console.log(this.total);
      console.log(this.paginas);
      console.log(this.limit);

      const data = await query(
        collection(db, "obras"),
        orderBy("author"),
        limit(this.limit)
      );
      const querySnapshot = await getDocs(data);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
        console.log(doc);
      });
    },
    async siguiente() {
      const q = await query(collection(db, "obras"), orderBy("author"));
      const documentSnapshots = await getDocs(q);
      const firstVisible =
        documentSnapshots.docs[this.limit * (this.page - 1) - 1] || null;
      this.firstVisible = firstVisible;
      console.log(this.firstVisible);
      const next = await query(
        collection(db, "obras"),
        orderBy("author"),
        limit(this.limit),
        startAfter(this.firstVisible)
      );

      const querySnapshot = await getDocs(next);
      this.obras = [];
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
        console.log(doc);
      });
    },
  },
  mounted() {
    this.obtenerDatos();
  },
};
</script>

<style>
.text-center {
  text-align: center;
}
</style>
