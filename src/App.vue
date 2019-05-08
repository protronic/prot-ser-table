<template>
  <div id="app">
    <prot-table :data_url='data_url' :table_options="options"></prot-table>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue';
import Vue from 'vue';
import PouchDB from 'pouchdb-browser';
import ProtTable from 'prot-table';

let pouch = new PouchDB('http://prot-subuntu:5985/prot-table');

let url = new URL(location.href)
let query_string = url.search;
let search_params = new URLSearchParams(query_string);

let page = search_params.get('page');

export default {
  name: 'app',
  components: {
    ProtTable
  },
  mounted(){
    pouch.get(page)
      .then( response => {
        console.log(response);
        this.options = response;
      });
  },
  data(){
    return {
      data_url: 'data.json',
      options: {},
    };
  }
}
</script>

<style>
#app {
  /* font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px; */
}
</style>
