<template>
  <Header msg="Grafik sale taneczne"/>
  <button v-on:click="generateList">Generuj</button>
  <div v-if="seen">Wygenerowano: {{new Date()}}</div>
  <List v-for="item in list" v-bind:item="item" v-bind:key="item.key" />
</template>

<script>
import Header from './components/Header.vue'
import List from './components/List.vue'

const page = "https://api.saletaneczne.pl/inventory"
export default {
  name: 'App',
  components: {
    Header,
    List,
  },
  data: function() {
    return {
      seen: false,
      list: []
    }
  },
  methods: {
    generateList: async function () {
      this.seen = true
      this.list = []
      const date = "24.01.2022"
      const sh = 18

      await this.generateListForSpecificHour(date, sh)
      await this.generateListForSpecificHour(date, sh + 1)
      await this.generateListForSpecificHour(date, sh + 2)
      await this.generateListForSpecificHour(date, sh + 3)
    },
    generateListForSpecificHour: async function(date, sh) {
      await this.generateListForSpecificHourWithMinutes(date, sh.toString(), "00", (sh + 2).toString(), "00")
      await this.generateListForSpecificHourWithMinutes(date, sh.toString(), "30", (sh + 2).toString(), "30")
    },
    generateListForSpecificHourWithMinutes: function(date, sh, sm, fh, fm) {
      return new Promise((resolve) => fetch(`${page}?capacity=2&city=Wroc%C5%82aw&date=${date}&from=${sh}%3A${sm}&to=${fh}%3A${fm}&addressActive=true`)
        .then(response => response.json())
        .then(data => {
          if (data.totalCount > 0) {
            this.list.push({text: `${date} - ${sh}:${sm} - ${fh}:${fm}`})
          }
        })
        .finally(resolve())
    )
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
