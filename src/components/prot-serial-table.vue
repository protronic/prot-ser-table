<template>
  <div id="app">
    <prot-table :data_url='data_url' :table_options="options"></prot-table>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue';
// import Vue from 'vue';
import PouchDB from 'pouchdb-browser';
import ProtTable from 'prot-table/src/App';

let pouch = new PouchDB('http://prot-subuntu:5985/prot-table');

let url = new URL(location.href)
let query_string = url.search;
let search_params = new URLSearchParams(query_string);

let page = search_params.get('page');

export default {
  name: 'protSer',
  components: {
    ProtTable
  },
  mounted(){
    pouch.get(page)
      .then( response => {
        console.log(response);
        this.options = this.deserialize_options(JSON.stringify(response));
        console.log(this.options)
      });
  },
  props: {
    data_url: String,
  },
  data(){
    return {
      // data_url_d: 'data.json',
      options: {},
    };
  },
  methods: {
    deserialize_options(json_options){
      let regex = /function\((.*)\)\{((.*\n{0,1})*)\}/;
      return JSON.parse(json_options, function(key, value){
        if(regex.test(value)){
          let match = regex.exec(value);
          return new Function(...(match[1].split(',').map( str => (str.trim()))), match[2])
        }
        else{
          return value;
        }
      });
    },
    
  },
  // watch: {
  //   data_url: {
  //     immediate: true,
  //     handler(){
  //       console.log(this.data_url)
  //       this.data_url_d = this.data_url
  //     }
  //   }
  // }
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
