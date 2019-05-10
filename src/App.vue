<template>
  <div id="app">
    <prot-ser :data_url='data_url' :height="height"></prot-ser>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue';
import Vue from 'vue';
import protSer from './components/prot-serial-table';
 
export default {
  name: 'app',
  components: {
    protSer
  },
  props: {
    data_url: {
      type: String,
      default: 'data.json'
    },
    height: {
      type: [Number, String],
      default: "auto"
    }
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
