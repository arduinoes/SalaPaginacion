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
  <div class="d-flex justify-content-center">
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li class="d-flex page-item me-5">
          <button
            :disabled="activoAnterior"
            class="page-link"
            @click.prevent="anterior"
          >
            Anterior
          </button>
        </li>
        <li class="d-flex page-item me-5">
          <button
            :disabled="activoSiguiente"
            class="page-link"
            @click.prevent="siguiente"
          >
            Siguente
          </button>
        </li>
      </ul>
    </nav>
  </div>
  {{ contador }}
</template>

<script>
import Navbar from "../components/Navbar.vue";
import BaseTarjeta from "../components/BaseTarjeta.vue";
import Pagination from "../components/Pagination.vue";
import {
  collection,
  getDocs,
  query,
  startAt,
  startAfter,
  orderBy,
  limit,
  endBefore,
  limitToLast,
} from "firebase/firestore";
import { db } from "../main";
export default {
  name: "ListaObras",
  components: {
    Navbar,
    BaseTarjeta,
    Pagination,
  },
  data() {
    return {
      contador: "",
      total: "",
      limit: 4,
      paginas: "",
      page: 1,
      lastVisible: "",
      primerDato: "",
      firstVisible: "",
      activoSiguiente: false,
      activoAnterior: false,
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
    async tomarTotal() {
      const totalDatos = query(collection(db, "obras"));
      const documentSnapshots = await getDocs(totalDatos);
      const total = documentSnapshots.docs.length;
      this.total = total;
      this.paginas = Math.ceil(this.total / this.limit);
    },
    async obtenerDatos() {
      const first = query(collection(db, "obras"), orderBy("author"), limit(this.limit));
      this.tomarTotal();
      const documentSnapshots = await getDocs(first);
      const total = documentSnapshots.docs.length;
      this.total = total;
      const lastVisible =
        documentSnapshots.docs[documentSnapshots.docs.length - 1] || null;
      const firstVisible = documentSnapshots.docs[0] || null;
      this.firstVisible = firstVisible;
      this.lastVisible = lastVisible;
      console.log(this.total);

      const querySnapshot = await getDocs(first);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
        console.log(obra);
      });
    },
    async botonSiguiente() {
      const next = query(
        collection(db, "obras"),
        orderBy("author"),
        limit(this.limit),
        //offset(2),
        startAfter(this.lastVisible)
      );
      const documentSnapshots = await getDocs(next);
      const total = documentSnapshots.docs.length;
      this.total = total;
      const lastVisible =
        documentSnapshots.docs[documentSnapshots.docs.length - 1] || null;
      this.lastVisible = lastVisible;
      const firstVisible = documentSnapshots.docs[0] || null;
      this.firstVisible = firstVisible;
      console.log("last", lastVisible);

      this.obras = [];
      const querySnapshot = await getDocs(next);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
        console.log("page", this.page);
      });
    },
    async botonAnterior() {
      const back = query(
        collection(db, "obras"),
        orderBy("author"),
        limitToLast(this.limit),
        endBefore(this.firstVisible)
      );

      const documentSnapshots = await getDocs(back);
      const lastVisible =
        documentSnapshots.docs[documentSnapshots.docs.length - 1] || null;
      this.lastVisible = lastVisible;
      const firstVisible = documentSnapshots.docs[0] || null;
      this.firstVisible = firstVisible;

      this.obras = [];
      const querySnapshot = await getDocs(back);
      querySnapshot.forEach((doc) => {
        let obra = doc.data();
        obra.id = doc.id;
        this.obras.push(obra);
      });
    },
      siguiente() {
      this.activoAnterior = false;
      if (this.page != this.paginas) {
        this.botonSiguiente();
        this.page = this.page + 1;
      } else {
        this.activoSiguente = true;
      }
    },
    anterior() {
      this.activoSiguente = false;
      if (this.page != 1) {
        this.botonAnterior();
        this.page = this.page - 1;
      } else {
        this.activoAnterior = true;
      }
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
