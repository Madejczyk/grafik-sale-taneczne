<template>
  <Header msg="Grafik sale taneczne"/>
  <div>
    <datepicker v-model="selectedDate" range :enableTimePicker="false" locale="pl" :format="inputFormat" placeholder="Domyślnie dziś" />
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

const PAGE = "https://api.saletaneczne.pl/inventory"
const FIRST_HOUR = 18
const LAST_HOUR = 21

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
    inputFormat: function (dateProxy) {
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
    onBegin: function() {
      this.list = []
      this.countResponse = 0
    },
    generateList: function () {
      this.onBegin()
      let sd = this.selectedDate;
      let ed = null
      if (isProxy(this.selectedDate)) {
        const dateObj = toRaw(this.selectedDate)
        sd = dateObj[0]
        ed = dateObj[1]
      }
      const endRangeExists = ed !== null
      if (endRangeExists) {
        console.log()
      } else {
        this.generateListForSpecificDay(sd)
      }
    },
    generateListForSpecificDay: function(sd) {
      this.maxResponse = (LAST_HOUR - FIRST_HOUR + 1) * 2
      for (let i = FIRST_HOUR; i <= LAST_HOUR; i++) {
        this.generateListForSpecificHour(this.getDateFormat(sd), i)
      }
    },
    generateListForSpecificHour: function(date, sh) {
      this.generateListForSpecificHourWithMinutes(date, sh.toString(), "00", (sh + 2).toString(), "00")
      this.generateListForSpecificHourWithMinutes(date, sh.toString(), "30", (sh + 2).toString(), "30")
    },
    generateListForSpecificHourWithMinutes: function(date, sh, sm, fh, fm) {
      fetch(`${PAGE}?capacity=2&city=Wroc%C5%82aw&date=${date}&from=${sh}%3A${sm}&to=${fh}%3A${fm}&addressActive=true`)
        .then(response => response.json())
        .then(info => {
          this.countResponse++
          if (info.totalCount > 0) {
            const price = info.data[0].priceData.price / 100;
            const text = `${date} - ${sh}:${sm} - ${fh}:${fm} (${price} zł)`
            this.list.push({text})
          }

          if (this.isFinish()) {
            this.onFinish()
          }
        })
    },
    isFinish: function() {
      return this.countResponse === this.maxResponse
    },
    onFinish: function() {
      this.list.sort((a, b) => (a.text > b.text) ? 1 : -1)
      this.maxResponse = 0
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

.dp__input {
  border: 2px solid #4179E0;
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
  margin: 1em
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
