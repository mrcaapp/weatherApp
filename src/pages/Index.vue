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
          <q-icon name="my_location"/>
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
            <span>Sunrise</span><br>
            {{timeSunrise}}
          </div>
          <div class="col-6">
            <span>Sunset</span><br>
            {{timeSunset}}
          </div>
        </div>
      </div>
      <div class="col text-center q-pb-lg">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`">
      </div>
    </template>
    <template v-else>
      <div class="col text-center text-white">
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
      search: '',
      weatherData: null,
      timeSunset: null,
      timeSunrise: null,
      lat: null,
      long: null,
      apiURL: 'https://api.openweathermap.org/data/2.5/weather',
      apiKEY: '6415963eb5a25d009c6224989bd84f8e'
    }
  },
  // mounted () {
  //   this.getLocation()
  // },
  computed: {
    bgClass() {
      if(this.weatherData){
        if(this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      } else {
        return ''
      } 
    }
  },
  methods: {
    getLocation () {
      navigator.geolocation.getCurrentPosition (p => {
        this.lat = p.coords.latitude
        this.long = p.coords.longitude
        this.getWeather()
      })
    },
    getWeather () {
      axios(`${ this.apiURL }?lat=${ this.lat }&lon=${ this.long }&&appid=${ this.apiKEY }&units=metric`)
        .then(r =>{
          this.weatherData = r.data
          this.timeSunrise = this.formatHour(r.data.sys.sunrise)
          this.timeSunset = this.formatHour(r.data.sys.sunset)
          console.log(this.weatherData)
        })
    },
    getWeatherBySearch () {
      axios(`${ this.apiURL }?q=${ this.search }&appid=${ this.apiKEY }&units=metric`)
        .then(r =>{
          this.weatherData = r.data
          this.timeSunrise = this.formatHour(r.data.sys.sunrise)
          this.timeSunset = this.formatHour(r.data.sys.sunset)
        })
    },
    formatHour (hour) {
      let date = new Date(hour*1000)
      return date.getHours(date) + ':' + date.getMinutes(date)
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
      background: linear-gradient(to bottom, #00b4db, #0083b0)
  .degree
    top: -53px
  .skyline
    flex: 00 100px
    background: url(../assets/skyline.png)
    background-size: contain
    background-position: center bottom 
</style>