<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-md q-px-md">
      <q-input 
        color="white" 
        v-model="location" 
        label="Search a location" 
        label-color="white"
        @keyup.enter="getWeatherBySearch"
        standout
        clearable
      >
        <template v-slot:prepend>
          <q-icon 
            @click="getLocation"
            name="place" 
            color="white" 
          />
        </template>
      </q-input>
    </div>

    <template v-if="hasData">
      <div class="col text-white text-center">
        <div class="q-pt-sm text-h3 text-weight-light">
          {{ currWeather.data.name }}
        </div>
        <div class="text-h4 text-weight-light">
          {{ currWeather.data.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(currWeather.data.main.temp) }}</span>
          <span class="text-h2 relative-position degree">&deg;</span>
        </div>
        <div class="col text-center">
          <img :src="`http://openweathermap.org/img/wn/${ currWeather.data.weather[0].icon }@2x.png`">
        </div>
        <div class="col q-ma-sm">
          <img v-if="bgClass==='bg-night'" class="silh col night" :src="`gondolier-29216_640.png`">
          <img v-else class="silh col day" :src="`fisherman-3654878_640.png`">
        </div>
      </div>
      
    </template>
    <!-- <div class="col boat"> -->
    <!-- </div> -->
    <template v-else>
      <div class="col column text-white text-center">
        <div class="col text-h2 text-weight-thin">
          Unit<br/>
          Weather
        </div>
        <q-btn class="col" flat @click="getLocation" >
          <q-icon left size="3em" name="pin_drop" />
          <div>Check current location</div>
        </q-btn>
      </div>
    </template>

    <div class="col details" v-if="hasData">
      <q-icon class="col q-ml-lg text-white text-center" size="lg" name="expand_more"></q-icon>
      <h4 class="col text-center text-white">
        Weekly Forecast
      </h4>
      <div class="row text-center text-h5">
        <div class="col forecast-card">
          <q-card class="my-card bg-purple forecast-card" v-for="day in weeklyWeather">
            <q-card-section class="bg-primary text-white">
              <div class="text-h6"> {{ day.dt }} </div>
            </q-card-section>

            <q-separator />

            <q-card-section>
              <div>{{ day.weather[0].main}}</div>
              <img :src="`http://openweathermap.org/img/wn/${ day.weather[0].icon }@2x.png`">
            </q-card-section>
          </q-card>
        </div>
      </div>

    </div>

  </q-page>
</template>

<script>
import { defineComponent, ref, reactive, computed } from 'vue';
import { useQuasar } from 'quasar'
import { api } from 'boot/axios'

export default defineComponent({
  name: 'PageIndex',

  setup() {
    let location = ref('')
    let currWeather = reactive({data: {}})
    let weeklyWeather = reactive([])
    let hasData = ref(currWeather.data.base ? true : false)
    const $q = useQuasar()
    const latitude = ref(null)
    const longitude = ref(null)
    // const apiKey = ref('1998301f5ecfbb123dc2a6901e52ce68')
    // const apiUrl = ref('api.openweathermap.org/data/2.5/weather')
    const bgClass = computed(() => {
      if(hasData.value){
        if(currWeather.data.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }
        else{
          return 'bg-day'
        }
      }
    });

    // functions

    function getLocation(){
      $q.loading.show()

      if($q.platform.is.electron){
        api.get("https://freegeoip.app/json/").then(response => {
          latitude.value = response.data.latitude
          longitude.value = response.data.longitude
          getWeatherByCoords()
        })
      }
      else{
        navigator.geolocation.getCurrentPosition(
          position => {
            latitude.value = position.coords.latitude
            longitude.value = position.coords.longitude
            getWeatherByCoords()
            getweeklyWeather()
          }
        )
        
      }

    }

    function getWeatherByCoords(){
      $q.loading.show()
      api.get(`${ process.env.API_URL }?lat=${ latitude.value }&lon=${ longitude.value }&appid=${ process.env.API_KEY }&units=metric`)
      .then(response => {  
          currWeather.data = response.data
      })

      
    }

    function getweeklyWeather(){
      api.get(`${ process.env.ONE_CALL_URL }?lat=${ latitude.value }&lon=${ longitude.value }&appid=${ process.env.API_KEY }&units=metric`)
        .then(response => {
          weeklyWeather = response.data.daily
          console.log(weeklyWeather)
          hasData.value = true
          $q.loading.hide()
        })
    }

    function getWeatherBySearch(){
      $q.loading.show()
      api.get(`${ apiUrl.value }?q=${ location.value }&appid=${ apiKey.value }&units=metric`)
      .then(response => {  
          currWeather.data = response.data
          getweeklyWeather()
          hasData.value = true
          $q.loading.hide()
      })
    }

    return {
      location, 
      currWeather,
      weeklyWeather,
      hasData,
      bgClass,

      getLocation,
      getWeatherBySearch

    }
  }
});

</script>

<style lang="scss">
  .q-page {
    background: linear-gradient(to bottom, #36d1dc, #5b86e5);
    &.bg-night {
      background: linear-gradient(to bottom, #141e30, #243b55);
    }&.bg-day {
      background: linear-gradient(to bottom, #36d1dc, #5b86e5);
    }
  }
  .degree {
    top: -35px
  }
  .silh {
    &.day {
      width: 150px;
      height: 150px;
      opacity: 75%;
    }
    &.night {
      width: 100%;
      opacity: 100%;
    }
  }
  img {
    max-width: 100%;
    max-height: 100%;
  }
  .details {
    min-height: 100%; //add min-height
    max-width: 100%;
    height: auto;
    background-color: red; //changed so you can see the container better
    overflow: hidden; //add to clear floats
    margin: 50% 0 0; //add margin from top
  }
  .forecast-card {
    max-width: 150px;
  }

</style>
