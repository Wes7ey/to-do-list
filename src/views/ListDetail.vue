<script>
import { listApiMixin } from "@/api/todolists";
import { toDoItemsApiMixin } from "@/api/todoitems";
import { dateFormater } from "@/mixin/dateFormater";

import modalDetail from "@/components/modal/detail-item.vue";
import ModalNewList from "@/components/modal/new-item.vue";
import Loading from "@/components/Loading.vue";
import confirmDelete from "@/components/modal/alertDelete.vue";
import ModalTasks from "@/components/ModalTasks.vue";

import Error from "@/components/Error.vue";

export default {
  components: {
    modalDetail,
    ModalNewList,
    Loading,
    confirmDelete,
    ModalTasks,
    Error,
  },
  mixins: [listApiMixin, toDoItemsApiMixin, dateFormater],
  data() {
    return {
      items: "",
      itemDetailInfos: {},
      showModalDetail: false,
      showNewItemForm: false,

      loading: false,

      selected: [],
      seeAllTasks: "",
      showModalDelete: false,
      titleModalDelete: "",

      showInfoTask: false,
      nameList: '',

      ErrorModal: false,
      errorMessage: '',

    };
  },
  async mounted() {
    this.loading = true;
    this.getItems();
  },
  methods: {
    async getItems() {
      const id = this.$route.params.id;
      try {
        const { data } = await this.detailList(id);
        this.items = data.items;
        this.nameList = data.title;
        this.loading = false;
      } catch (err) {
        this.loading = false;
        this.errorMessage = err.response.data.message
        this.ErrorModal = true
      } finally {
        this.loading = false
      }
    },

    newItem(title, deadline) {
      const id = this.$route.params.id;
      this.handleWithResponse(this.createItem(id, title, deadline));
      this.showNewItemForm = false;
    },

    handleResolveItem(id) {
      /* this.handleWithResponse(
        Promise.all(this.selected.map((el) => this.ResolveItem(el)))
      ); */
      this.handleWithResponse(this.ResolveItem(id))
      this.selected = [];
    },

    handledOpenDeleteModal(id, title) {
      this.titleModalDelete = [id, title];
      this.showModalDelete = true;
    },

    handleDeleleItem(id) {
      this.showModalDelete = false
      this.handleWithResponse(this.deleteItem(id));

    },

    async handleShowDetail(id) {
      const res = await this.handleWithResponse(this.DetailItem(id));
      this.itemDetailInfos = res.data;
      this.showModalDetail = true;
    },



    /* FUNÇÂO QUE TRATA RETORNO DA API, RECEBE UMA PROMISSE E CONFORME RESULTADO EXECUTA AÇÂO */
    async handleWithResponse(promise) {
      try {
        this.loading = true;
        return await promise;
      } catch (err) {
        this.loading = false
        this.ErrorModal = true
        this.errorMessage = err.response.data.message
      } finally {
        this.getItems();
      }
    },

    /* METODO DE ESTILIZAÇÂO */
    bgColor(a) {
      return a == true ? "bg-light-green-lighten-2" : "bg-red-lighten-3";
    },

    /* ORGANIZAR A LISTA CONFORME DEADLINE */
    organizeDeadlineDate(state) {
      if (!this.items) return "";
      const ordened = this.items.sort(
        (a, b) => new Date(a.deadline) - new Date(b.deadline)
      );
      const undone = ordened.filter((elemento) => elemento.done == false);
      const done = ordened.filter((elemento) => elemento.done == true);

      if (state) return undone.concat(done);
      return undone;
    },
  },
};
</script>

<template>
  <div class="w-100 bg-white d-flex justify-space-between">

    <v-row>
      <v-col cols="4">
        <v-btn color=black @click="this.$router.go(-1)" variant="plain">
          <span class="material-symbols-outlined">
            arrow_back
          </span>
        </v-btn>
      </v-col>

      <v-col cols="4" class="d-flex justify-center">
        <v-btn color=black @click="showNewItemForm = true" variant="plain"> NEW ITEM </v-btn>
      </v-col>


      <v-col cols="4" class="d-flex justify-end">
        <div class="d-flex flex-row justify-center align-center">
          <h5>SEE ALL TASKS:</h5>
          <v-checkbox v-model="seeAllTasks" class="d-flex" />
        </div>
      </v-col>
    </v-row>
  </div>

  <v-card>
    <v-card-title class="elevation-0 d-flex justify-center">
      {{ nameList }}
    </v-card-title>
  </v-card>

  <v-alert v-if="organizeDeadlineDate().length == 0" type="success" title="Congrats!" class="w-75 mx-auto" closable
    text="There are no open tasks in this list!" variant="tonal">
  </v-alert>

  <div class="w-75 mx-auto d-flex flex-column justify-center align-center elevation-0">
    <div v-for="item in organizeDeadlineDate(seeAllTasks)">
      <ModalTasks :item=item @delete-item="handledOpenDeleteModal" @resolve-item="handleResolveItem"
        @detail-item="handleShowDetail"></ModalTasks>
    </div>
  </div>

  <!-- MODAL DE NOVO ITEM NA LISTA -->
  <ModalNewList v-if="showNewItemForm" @new-item="newItem" @close="this.showNewItemForm = false"></ModalNewList>

  <!-- MODAL DE DETALHE DO ITEM -->
  <modalDetail v-if="showModalDetail" :listName="nameList" :infos="itemDetailInfos"
    @closeModal="this.showModalDetail = false"></modalDetail>

  <!-- MODAL DE LOADING -->
  <Loading v-if="loading"></Loading>

  <!-- MODAL DE ALERTA PARA DELEÇÃO -->
  <confirmDelete v-if="showModalDelete" :title="titleModalDelete" @closed-modal="this.showModalDelete = false"
    @confirm-modal="handleDeleleItem">
  </confirmDelete>

  <Error v-if="ErrorModal" :error="errorMessage" @close="ErrorModal = false"></Error>
</template>

<style>
.c-pointer {
  cursor: pointer;
}
</style>
