<template>
  <div>
    <div id="side-menu">
      <form @submit.prevent="search()">
        <div class="controle">
          <label for="searchField">Generic search</label>
          <input id="searchField" v-model="filter" />
        </div>
        <div class="controle">
          <label for="searchTitle">Title</label>
          <input id="searchTitle" v-model="filterTitle" />
        </div>
        <div class="controle">
          <input type="checkbox" name="whiteWine" value="Vin blanc" v-model="whiteWine" />White Wine
          <br />
          <input type="checkbox" name="redWine" value="Vin rouge" v-model="redWine" />Red Wine
        </div>
        <button id="search" @click="search()">Search</button>
      </form>
    </div>
    <div id="wines-list">
      <li class="wines" v-for="wine of wines">
        <meu-painel :title="wine.title" :uri="wine.uri" :thumbnail="wine.raw.tpthumbnailuri" :category="wine.raw.tpcategorie"/>
      </li>
    </div>
  </div>
</template>

<script>
import Painel from "../components/Painel.vue";
import axios from "axios";

export default {
  components: {
      "meu-painel" : Painel
  },
  data() {

    return {
      wines: [], 
      filter: "",
      filterTitle: "",
      whiteWine: "",
      redWine: "",
      mountedFilter: ""
    }
  },
  created() {
    this.search();
  },
  methods: {
    search() {
      let loader = this.$loading;
      loader.show();
      if(this.filter != "")
        this.mountedFilter = this.mountedFilter + this.filter;
      if(this.filterTitle != '')
        this.mountedFilter = this.mountedFilter + " @title=" + this.filterTitle;
      if(this.whiteWine == true && this.redWine == true) {
        this.mountedFilter = this.mountedFilter + " @tpcategorie=Vin rouge OR @tpcategorie=Vin blanc "
      } else if(this.whiteWine == true) {
        this.mountedFilter = this.mountedFilter + " @tpcategorie=Vin blanc "
      } else if(this.redWine == true) {
        this.mountedFilter = this.mountedFilter + " @tpcategorie=Vin rouge "
      }
      if(this.mountedFilter != '') {
        axios.post("http://localhost:8080/coveo/v1/coveo", {
          params: this.mountedFilter
        })
          .then(response => {
            this.wines = response.data,
            loader.hide();
          }, err => console.log('erro ' + err.data));
      } else {
        axios.post("http://localhost:8080/coveo/v1/coveo/all")
          .then(response => {
            this.wines = response.data,
            loader.hide();
          }, err => console.log("erro all " + err.data));
      }
      this.mountedFilter = "";
    }
  }
};
</script>
<style scoped>
  #side-menu {
    float: left;
    width: 30%;
  }

  #wines-list {
    width: 70%;
    float: right;
  }

  .wines {
    list-style-type: none;
  }

  .controle {
    text-align: left;
    margin-bottom: 10px;
  }
</style>