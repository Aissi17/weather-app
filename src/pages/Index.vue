<template>
  <q-page class="flex column" :class="bgClass">

  <div class="col q-pt-lg q-px-md"> 
    <q-input 
      v-model="search"
      @keyup.enter="getWeatherbySearch"
      placeholder="Search"
      dark
      borderless
      >
      <template v-slot:before>
        <q-icon
          name="my_location" 
          @click="getLocation"
        />
      </template>

      <template v-slot:append>
        <q-btn
          round
          dense
          flat
          icon="search" 
          @click="getWeatherbySearch"
          />
      </template>
    </q-input>
  </div>

  <template v-if="weatherData" >
      <div class="col text-white text-center">
      <div class="text-h4 text-weight-light">
        {{ weatherData.name }}
      </div>
      <div class="text-h6 text-weigth-light">
        {{ weatherData.weather[0].main }}
      </div>
      <div class="text-h1 text-weigth-thin q-my-lg relative-position">
        <span>{{ Math.round(weatherData.main.temp)}}</span>
        <span class="text-h4 relative-position degree">&deg;c</span>
      </div>
    </div>

    <div class="col text-center">
      <img :src="`http://openweathermap.org/img/wn/${ weatherData.weather[0].icon }.png`" alt="Bill">
    </div>
  </template>
 
  <template v-else >
    <div class="col column text-center text-white">
      <div class="col text-h2 text-weigth-thin">
          Weather <br> App
      </div>
      <q-btn
        @click="getLocation"
        class="col"
        flat

        >
        <q-icon left size="3em" name="my_location" />
        <div>Find my location</div>
      </q-btn>
    </div>
  </template>


  <div class="col skiline">

  </div>

  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data (){
    return {
      search : '',
      weatherData : null,
      lat : null,
      long : null,
      apiURL : "http://api.openweathermap.org/data/2.5/weather",
      apiKey : "8f70c49306088bc0d3de9f9f42b7d0a6"


    }
  },
  computed : {
    bgClass(){
      if(this.weatherData){
        if(this.weatherData.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }
        else{
          return 'bg-day'
        }
      }
    }
  },
  methods : {
    getLocation(){
      this.$q.loading.show()

      if(this.$q.platform.is.electron){
        this.$axios.get('https://freegeoip.app/json/')
          .then((res)=>{
            this.lat = res.data.latitude
            this.long = res.data.longitude
            this.getWeatherbyCoords()
            this.$q.loading.hide()
          })
      }
      else{

        navigator.geolocation.getCurrentPosition(
        position => {
           this.lat = position.coords.latitude
           this.long = position.coords.longitude
           this.getWeatherbyCoords()
           this.$q.loading.hide()
        }
      )
      }

      
    },
    getWeatherbyCoords(){
      this.$q.loading.show()
      this.$axios(`${this.apiURL}?lat=${this.lat}&lon=${this.long}&appid=${this.apiKey}&units=metric`)
        .then((res)=>{
          this.weatherData = res.data
          this.$q.loading.hide()
        })
    },
    getWeatherbySearch(){
      this.$q.loading.show()
      this.$axios(`${this.apiURL}?q=${this.search}&appid=${this.apiKey}&units=metric`)
        .then((res)=>{
          this.weatherData = res.data
          this.$q.loading.hide()
        })
    }
  }
}
</script>

<style lang='sass'>
  .q-page
    background: linear-gradient(to bottom, #136a8a, #267871)
    &.bg-night
      background : linear-gradient(to bottom, #232526, #414345)
    &.bg-day 
      background : linear-gradient(to bottom, #00b4db, #0083b0)
  .degree
    top : -44px
  .skiline 
    flex : 0 0 130px
    background : url(../statics/town.png)
    background-size : contain 
    background-position : center bottom
</style>