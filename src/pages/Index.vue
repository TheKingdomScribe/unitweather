<template>
  <q-page class="flex column" :class="bgClass">
    <div class=" col q-pt-lg q-px-md">
      <q-input 
        color="white" 
        v-model="location" 
        label="Search a location" 
        label-color="white"
        @keyup.enter="getWeatherBySearch"
        filled
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
        <div class="text-h3 text-weight-light">
          {{ weatherData.data.name }}
        </div>
        <div class="text-h4 text-weight-light">
          {{ weatherData.data.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.data.main.temp) }}</span>
          <span class="text-h2 relative-position degree">&deg;</span>
        </div>
      </div>
      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${ weatherData.data.weather[0].icon }@2x.png`">
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
    let weatherData = reactive({data: {}})
    let hasData = ref(weatherData.data.base ? true : false)
    const $q = useQuasar()
    const latitude = ref(null)
    const longitude = ref(null)
    const apiKey = ref('1998301f5ecfbb123dc2a6901e52ce68')
    const apiUrl = ref('api.openweathermap.org/data/2.5/weather')
    const bgClass = computed(() => {
      if(hasData.value){
        if(weatherData.data.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }
        else{
          return 'bg-day'
        }
      }
    });


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
          }
        )
        
      }

    }

    function getWeatherByCoords(){
      $q.loading.show()
      api.get(`${ apiUrl.value }?lat=${ latitude.value }&lon=${ longitude.value }&appid=${ apiKey.value }&units=metric`)
      .then(response => {  
          weatherData.data = response.data
          hasData.value = true
          $q.loading.hide()
      })         
    }

    function getWeatherBySearch(){
      $q.loading.show()
      api.get(`${ apiUrl.value }?q=${ location.value }&appid=${ apiKey.value }&units=metric`)
      .then(response => {  
          weatherData.data = response.data
          $q.loading.hide()
      })
    }

    return {
      location, 
      weatherData,
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
    top:-35px
  }
</style>
