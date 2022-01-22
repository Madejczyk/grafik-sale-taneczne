<template>
  <Header msg="Grafik sale taneczne"/>
  <button v-on:click="generateList">Generuj</button>
  <List v-for="item in list" v-bind:item="item" v-bind:key="item.key" />
</template>

<script>
import Header from './components/Header.vue'
import List from './components/List.vue'

export default {
  name: 'App',
  components: {
    Header,
    List,
  },
  data: function() {
    return {
      list: []
    }
  },
  methods: {
    generateList: function () {
      this.list.push({text: "Test"})
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
button {
  background-color: white;
  color: black;
  border: 2px solid #4C0013;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  transition-duration: 0.4s;
  border-radius: 4px;
}
button:hover {
  background-color: #4C0013;
  color: white;
}
</style>
