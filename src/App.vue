<template>
  <Header msg="Grafik sale taneczne"/>
  <div>
    <input type="text" v-model="date">
  </div>
  <button v-on:click="generateList">Generuj</button>
  <div class="loading" v-if="isLoading()"><rotate-square2></rotate-square2></div>
  <List v-for="item in list" v-bind:item="item" v-bind:key="item.key" />
</template>

<script>
import {RotateSquare2} from 'vue-loading-spinner'
import Header from './components/Header.vue'
import List from './components/List.vue'

const page = "https://api.saletaneczne.pl/inventory"
export default {
  name: 'App',
  components: {
    RotateSquare2,
    Header,
    List,
  },
  data: function() {
    const d = new Date()
    let day = d.getDate();
    if (day.toString().length === 1) {
      day = "0" + day
    }
    let month = d.getMonth() + 1;
    if (month.toString().length === 1) {
      month = "0" + month
    }
    return {
      date: `${day}.${month}.${d.getFullYear()}`,
      end: false,
      list: [],
      maxResponse: 0,
      countResponse: 0
    }
  },
  methods: {
    isLoading: function () {
      return this.maxResponse > 0
    },
    generateList: function () {
      this.list = []
      const date = this.date
      const sh = 18
      const lh = 21
      this.maxResponse = (lh - sh + 1) * 2
      this.countResponse = 0

      for (let i = sh; i <= lh; i++) {
        this.generateListForSpecificHour(date, i)
      }
    },
    generateListForSpecificHour: function(date, sh) {
      this.generateListForSpecificHourWithMinutes(date, sh.toString(), "00", (sh + 2).toString(), "00")
      this.generateListForSpecificHourWithMinutes(date, sh.toString(), "30", (sh + 2).toString(), "30")
    },
    generateListForSpecificHourWithMinutes: function(date, sh, sm, fh, fm) {
      fetch(`${page}?capacity=2&city=Wroc%C5%82aw&date=${date}&from=${sh}%3A${sm}&to=${fh}%3A${fm}&addressActive=true`)
        .then(response => response.json())
        .then(data => {
          this.countResponse++
          if (data.totalCount > 0) {
            this.list.push({text: `${date} - ${sh}:${sm} - ${fh}:${fm}`})
          }
          if (this.countResponse === this.maxResponse) {
            this.list.sort((a, b) => (a.text > b.text) ? 1 : -1)
            this.maxResponse = 0
          }
        })
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

input[type=text] {
  padding: 12px 20px;
  margin: 8px 0;
  box-sizing: border-box;
  border: 2px solid #4179E0;
  border-radius: 4px;
}
button {
  background-color: white;
  color: black;
  border: 2px solid #4179E0;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  transition-duration: 0.4s;
  border-radius: 4px;
}
button:hover {
  background-color: #4179E0;
  color: white;
}
.loading {
  display: flex;
  justify-content: center;
  align-content: center;
}
.spinner::after {
  background: #d7bdff !important
}
</style>
