<template>
  <div>
    <prot-table ref="table" :display_options="displayoptions"></prot-table>
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
        
        this.fetch_data()

        this.$set(op, 'headerStyles', { 
          'text-align': 'left',
          'font-weight': 'bold'
        });

        this.$set(op, 'bodyStyles', {
          'text-align': 'left'
        });

        // this.$set(op, 'pagination', {
        //   'activ': true,
        //   'rows': 300
        // })

        // this.$set(op, 'headerDef', this.get_columns(this.data));
        
        this.$set(op, 'tableStyles', {
          'grid-gap': '2px'
        })
        
        this.$set(op, 'formatter', {
          active: true,
          externelFunction: undefined,
          options: {
            ...op.formatter,
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
            },
            "edit": (value, index, row) => {
              return `<a href="http://prot-subuntu:8080/prot-wiki/Wiki.jsp?page=Formly&formular=${row['#form']}&model=${row['#id']}">edit</a>`;
            },
          }
        });

        this.options = Object.assign({}, this.options, op);  
        // this.$refs.table.table_options = op;

        this.fetch_legend();
      });
  },
  props: {
    data_url: String,
    height: [Number, String],
    displayoptions: String,
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
      data: [],
    };
  },
  methods: {
    fetch_data(){
      if(this.data_url)
        fetch(this.data_url)
          .then( response => {
            return response.json()
          })
          .then( response => {
            if(response[0]['#id'] && response[0]['#form']){
              for(let i = 0; i < response.length; i++){
                response[i].edit = '';
              }
            }
            this.data = response;
            this.$set(this.options, 'headerDef', this.get_columns(response))
            this.$refs.table.table_options = this.options;
            this.$refs.table.data = response;
            console.log({data: response, options: this.options})
          })
          .catch(err => (console.error(err)));
    },
    get_columns(data){
      let columns = {};
      for(let key in data[0]){
        if(key !== '#id' && key !== '#form'){
          columns[key] = {displayName: key}
        }
      }
      if(data[0]['#id'] && data[0]['#form']){
        columns['edit'] = {displayName: 'edit'}
      }
      return columns;

    },
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
            if (this.options.formatter.options){
              if (this.options.formatter.options[key])
                this.$set(this.options.formatter.options, key, (value, rowIndex, row) => {
                  return `<a href="${links[key]}${value}">${this.options.formatter.options[key](value, rowIndex, row)}</a>`;
                });
              else {
                this.$set(this.options.formatter.options, key, (value) => {
                  return `<a href="${links[key]}${value}">${value}</a>`
                });
              }
            }
            // else{
            //   this.options.formatter.options = {}
            //   this.$set(this.options.formatter.options, key, (value) => {
            //     return `<a href="${links[key]}${value}">${value}</a>`
            //   });
            // }
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
