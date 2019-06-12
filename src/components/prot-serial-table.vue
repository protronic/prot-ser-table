<template>
  <div>
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
        return this.deserialize_options(JSON.stringify(response));
      })
      .catch( () => {
        console.error('Could not find page - style - document.');
        return {};
      })
      .then( op =>{
        if (this.height){
          this.$set(op, 'height', this.height);
        }
        // defaults:
        
        this.$set(op, 'headerStyles', { 
          'text-align': 'left',
          'font-weight': 'bold'
        });

        this.$set(op, 'bodyStyles', {
          'text-align': 'left'
        });

        this.$set(op, 'pagination', {
          'activ': true,
          'rows': 300
        })

        op.formatter = Object.assign({}, op.formatter, {
          "#all": (value, index, row) => {
            if(value === undefined){
              return value;
            }
            else if (value.length > 0 && value[0] === '-'){
              return `<span style="color: red;">${value}</span>`
            }
            else{
              return value;
            }
          }
        })

        this.options = Object.assign({}, this.options, op);  
        this.$refs.table.table_options = op;

        this.fetch_legend();
      });
  },
  props: {
    data_url: String,
    height: [Number, String],
    debug_con: {
      type: Object,
      default: () => ({
        log(){},
        error(){}
      })
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
    fetch_legend(){
      fetch('http://prot-subuntu:5985/ang_prot-wiki/prot-wiki_Legende')
        .then( response => (response.json()))
        .then( response => {
          console.log({response})
          let links = response.auskunftSchemaLinks;
          let tooltips = response.TooltipText;

          for(let key in links){
            if (this.options.formatter){
              if (this.options.formatter[key])
                this.$set(this.options.formatter, key, (value, rowIndex, row) => {
                  return `<a href="${links[key]}${value}">${this.options.formatters[key](value, rowIndex, row)}</a>`;
                });
              else {
                this.$set(this.options.formatter, key, (value) => {
                  return `<a href="${links[key]}${value}">${value}</a>`
                });
              }
            }
            else{
              this.options.formatter = {}
              this.$set(this.options.formatter, key, (value) => {
                return `<a href="${links[key]}${value}">${value}</a>`
              });
            }
          }
          this.$refs.table.table_options = this.options;
        })
        .catch( err => {
          console.error(err)
        })
    }
    
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
  // watch: {
  //   options(neu){
  //     this.$refs.table.table_options = this.options;
      
  //     console.log({value: neu, ref: this.$refs.table})
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
