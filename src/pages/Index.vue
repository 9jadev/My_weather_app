<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input bottom-slots @keyup.enter="getWeatherBySearch" v-model="search" placeholder="search" borderless="" dark>
        <template v-slot:before>
          <q-icon name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn round dense @click="getWeatherBySearch" flat icon="search" />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>
      
      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" />
      </div>
    </template>
    <template v-else> 
      <div class="col column text-white text-center">
        <div class="col text-h2 text-weight-thin">
          Solo<br>Weather
        </div>
        <q-btn class="col" @click="getLocation" flat>
            <q-icon left size="3em" name="my_location" />
            <div>Find my location</div>
        </q-btn>
      </div>
    </template>  

    <div class="col skyline">
    </div>

    <q-dialog v-model="alertNetwork">
      <q-card>
        <q-card-section>
          <div class="text-h6">whoop's</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          It's seems i can't talk to the internet now , check connection
        </q-card-section>
        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Close" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="WeatherError">
      <q-card>
        <q-card-section>
          <div class="text-h6">whoop's</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          It's seems like we can't find your location now , try again later
        </q-card-section>
        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Close" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data(){
    return {
      search: '',
      weatherData: null,
      lat: null,
      lon: null,
      alertNetwork: false,
      WeatherError: false,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: '62d591f438227a04dd80e165df803588'
    }
  },
  methods: {
    getLocation(){
      // console.log("ubik ihnk")
      this.$q.loading.show()
      if (this.$q.platform.is.electron){
        this.$axios(`https://freegeoip.app/json/`).then(response => {
          this.lat = response.data.latitude
          this.lon = response.data.longitude
          this.getWeatherByCoords()
        }).catch(error => {
          this.alertNetwork = true
          this.$q.loading.hide()
        }) 
      } else {
        navigator.geolocation.getCurrentPosition(position => {
        console.log('position', position)
        this.lat = position.coords.latitude
        this.lon = position.coords.longitude
        this.getWeatherByCoords()
      })
      }
    },
    getWeatherBySearch(){
      this.$q.loading.show()
      this.$axios(`${this.apiUrl}?q=${ this.search }&appid=${this.apiKey}&units=metric`).then(response => {
        // console.log('response', response)
        this.weatherData = response.data
        this.$q.loading.hide()
      }).catch(error => {
          this.WeatherError = true
          this.$q.loading.hide()
      }) 
    },  
    getWeatherByCoords(){
      this.$q.loading.show()
      this.$axios(`${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`).then(response => {
        // console.log('response', response)
        this.weatherData = response.data
        this.$q.loading.hide()
      }).catch(error => {
          this.alertNetwork = true
          this.$q.loading.hide()
      }) 
    }
  },
  computed: {
    bgClass() {
      if (this.weatherData){
        if (this.weatherData.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }else {
          return 'bg-day'
        }
      }
    }
  }
}
</script> 
<style lang="sass" scoped>
  .q-page
    background: #136a8a;  /* fallback for old browsers */
    background: -webkit-linear-gradient(to bottom, #267871, #136a8a);  /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(to bottom, #267871, #136a8a); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    &.bg-night
      background: linear-gradient(to bottom, #232526, #414345)
    &.bg-day
      background: linear-gradient(to bottom, #00b4db, #0083b0)  
  .degree
    top: -44px;
  .skyline
    flex: 0 0 100px
    background: url(../statics/skyline.png)
    background-size: contain
    background-position: center botton  
</style>
