<template>
  <Header msg="Grafik sale taneczne"/>
  <div>
    <datepicker v-model="selectedDate" range :enableTimePicker="false" locale="pl" :format="format" placeholder="Domyślnie dziś" />
  </div>  
  <button v-on:click="generateList">Generuj</button>
  <div class="loading" v-if="isLoading()"><rotate-square2></rotate-square2></div>
  <List v-for="item in list" v-bind:item="item" v-bind:key="item.key" />
</template>

<script>
import { isProxy, toRaw } from 'vue';
import {RotateSquare2} from 'vue-loading-spinner'
import Datepicker from 'vue3-date-time-picker';
import 'vue3-date-time-picker/dist/main.css'
import Header from './components/Header.vue'
import List from './components/List.vue'

const page = "https://api.saletaneczne.pl/inventory"
export default {
  name: 'App',
  components: {
    RotateSquare2,
    Datepicker,
    Header,
    List,
  },
  data: function() {
    return {
      selectedDate: new Date(),
      end: false,
      list: [],
      maxResponse: 0,
      countResponse: 0,
    }
  },
  methods: {
    format: function (dateProxy) {
      const date = toRaw(dateProxy)
      if (date[1] === null) {
        return this.getDateFormat(date[0]);
      }
      return `${this.getDateFormat(date[0])} - ${this.getDateFormat(date[1])}`;
    },
    isLoading: function () {
      return this.maxResponse > 0
    },
    getDateFormat: function (d) {
      let day = d.getDate();
      if (day.toString().length === 1) {
        day = "0" + day
      }
      let month = d.getMonth() + 1;
      if (month.toString().length === 1) {
        month = "0" + month
      }
      return `${day}.${month}.${d.getFullYear()}`
    },
    generateList: function () {
      this.list = []
      let sd = this.selectedDate;
      if (isProxy(this.selectedDate)) {
        sd = toRaw(this.selectedDate)[0]
      }
      const date = this.getDateFormat(sd)
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
  margin-top: 30px;
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
