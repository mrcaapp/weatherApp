<template>
  <q-page class="flex column " :class="bgClass">    
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keypress.enter="getWeatherBySearch"
        placeholder="buscar"
        borderless
        bottom-slots
        dark
      >
        <template v-slot:prepend>
          <q-icon name="my_location" @click="getLocation"/>
        </template>
        <template v-slot:append>
          <q-icon
            class="cursor-pointer"
            @click="getWeatherBySearch"
            name="search"
          />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{weatherData.name}}
        </div>
        <div class="text-h5 text-weight-light">
          {{weatherData.weather[0].main}}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{Math.round(weatherData.main.temp)}}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
        <div class="row text-weight-thin text-h6 q-pb-md">
          <div class="col-6">
            <span>Nascer do sol</span><br>
            {{timeSunrise}}
          </div>
          <div class="col-6">
            <span>Pôr do sol</span><br>
            {{timeSunset}}
          </div>
        </div>
      </div>
      <div class="col text-center q-pb-lg text-white text-weight-thin">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"><br>
        Velocidade do vento: {{(weatherData.wind.speed*2.913).toPrecision(3)}}Km/hr<br>
        {{hora}}
      </div>
    </template>
    <template v-else>
      <div class="col text-center text-white q-pb-md">
        <div class="col text-h2 text-weight-thin">
          Weather
        </div>
        <q-btn
          class="col q-mt-xl"
          @click="getLocation"
          flat
        >
          <q-icon left size="3em" name="my_location" />
          <div>Encontrar minha localização</div>
        </q-btn>
      </div>
    </template>

  </q-page>
</template>

<script>
import axios from 'axios'
export default {
  name: 'PageIndex',
  data () {
    return {
      hora: null,
      search: null,
      weatherData: null,
      timeSunset: null,
      timeSunrise: null,
      lat: null,
      long: null,
      apiURL: 'https://api.openweathermap.org/data/2.5/weather',
      apiKEY: '6415963eb5a25d009c6224989bd84f8e'
    }
  },
  mounted() {
    this.initTime()
  },
  computed: {
    bgClass() {
      if(this.weatherData){
        if(this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          if(this.weatherData.weather[0].main === 'Clouds') {
            return 'bg-day-clouds'
          } else if (this.weatherData.weather[0].main === 'Rain') {
            return 'bg-day-rain'
          } else {
            return 'bg-day'
          }
        }
      } else {
        return ''
      } 
    }
  },
  methods: {
    getHora () {
      var time = new Date()
      var hour = time.getHours()
      var min = time.getMinutes()
      var sec = time.getSeconds()

      if(hour < 10) hour = "0" + hour
      if(min < 10) min = "0" + min
      if(sec < 10) sec = "0" + sec
      
      this.hora = hour + ":" + min + ":" + sec
    },
    initTime () {
      setInterval(() => {this.getHora()}, 1000)
    },
    getLocation () {
      this.$q.loading.show()
      navigator.geolocation.getCurrentPosition (p => {
        this.lat = p.coords.latitude
        this.long = p.coords.longitude
        this.getWeather()
      })
      .catch(e => {
        this.$q.loading.hide()
        this.$q.notify({
          message: 'Localização não encontrada',
          color: 'grey-7'
        })
      })
    },
    getWeather () {
      this.$q.loading.show()
      axios(`${ this.apiURL }?lat=${ this.lat }&lon=${ this.long }&&appid=${ this.apiKEY }&units=metric`)
        .then(r => {
          this.weatherData = r.data
          this.timeSunrise = this.formatHour(r.data.sys.sunrise)
          this.timeSunset = this.formatHour(r.data.sys.sunset)
          this.$q.loading.hide()
          console.log(r)
        })
        .catch(e => {
          this.$q.loading.hide()
          this.$q.notify({
            message: 'Houve alguns problemas! Tente mais tarde',
            color: 'grey-7'
          })
        })
    },
    getWeatherBySearch () {
      if(this.search !== null && this.search !== ' '){
        this.$q.loading.show()
        axios(`${ this.apiURL }?q=${ this.search }&appid=${ this.apiKEY }&units=metric`)
          .then(r => {
            this.weatherData = r.data
            this.timeSunrise = this.formatHour(r.data.sys.sunrise)
            this.timeSunset = this.formatHour(r.data.sys.sunset)
            this.$q.loading.hide()
            console.log(r)
          })
          .catch(e => {
            this.$q.loading.hide()
            this.$q.notify({
              message: 'Não foi possivel encontrar o local',
              color: 'red'
            })
          })
      } else {
        this.$q.notify({
          message: 'Digite uma localização',
          color: 'grey-7'
        })
      }
    },
    formatHour (hour) {
      let time = new Date(hour*1000)
      return time.getHours(time) + ':' + time.getMinutes(time)
    }
  }
}
</script>
<style lang="sass">
  .q-page
    background: linear-gradient(to bottom, #556270, #ff6b6b)
    &.bg-night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #2980b9, #6dd5fa)
    &.bg-day-clouds
      background: linear-gradient(to bottom, #373b44, #4286f4)
    &.bg-day-rain
      background: linear-gradient(to bottom, #2a2c31, #1a4892)

  .degree
    top: -53px
  .skyline
    flex: 00 100px
    background: url(../assets/skyline.png)
    background-size: contain
    background-position: center bottom 
</style>
