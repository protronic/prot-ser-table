<template>
  <div id="app">
    <prot-table ref="table" :data_url='data_url'></prot-table>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue';
// import Vue from 'vue';
import PouchDB from 'pouchdb-browser';
import 'prot-table/dist/web-component/prot-table';

let pouch = new PouchDB('http://prot-subuntu:5985/prot-table');

let url = new URL(location.href)
let query_string = url.search;
let search_params = new URLSearchParams(query_string);

let page = search_params.get('page');

export default {
  name: 'protSer',
  components: {
    // ProtTable
  },
  mounted(){
    pouch.get(page)
      .then( response => {
        // console.log(response);
        return this.deserialize_options(JSON.stringify(response));
      })
      .catch( () => {
        debug_con.error('Could not find page - style - document.');
        return {};
      })
      .then( op =>{
        op.height = this.height;

        // defaults:

        op.headerStyles = {
          'text-align': 'left',
          'font-wright': 'bold'
        }

        op.bodyStyles = {
          'text-align': 'left'
        }

        this.options = op;  
        this.$refs.table.table_options = op;
        // console.log(this.options)
      });
  },
  props: {
    data_url: String,
    height: [Number, String],
    debug_con: {
      type: Object,
      default: {
        log(){},
        error(){}
      }
    }
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
