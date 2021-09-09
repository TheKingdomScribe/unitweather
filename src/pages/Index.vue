<template>
  <q-page class="flex column">
    <div class=" col q-pt-lg q-px-md">
      <q-input 
        color="white" 
        v-model="location" 
        label="Search a location" 
        label-color="white"
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

    <template v-if="weatherData==={}">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          Niagara
        </div>
        <div class="text-h6 text-weight-light">
          Cloudy
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>15</span>
          <span class="text-h2 relative-position degree">&deg;</span>
        </div>
      </div>
      <div class="col text-center">
        <img src="https://www.fillmurray.com/100/100" alt="img">
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
          <div>CHeck current location</div>
        </q-btn>
      </div>
    </template>
  </q-page>
</template>

<script>
import { defineComponent, ref, reactive } from 'vue';
import { useQuasar } from 'quasar'
import { api } from 'boot/axios'

export default defineComponent({
  name: 'PageIndex',

  setup() {
    
    let location = ref('')
    const $q = useQuasar()
    const weatherData = reactive({})
    const latitude = ref(null)
    const longitude = ref(null)
    const apiKey = ref('1998301f5ecfbb123dc2a6901e52ce68')
    const apiUrl = ref('api.openweathermap.org/data/2.5/weather')

    function getLocation(){
      navigator.geolocation.getCurrentPosition(
        position => {
          latitude.value = position.coords.latitude
          longitude.value = position.coords.longitude
          getWeather()
        }
      )
    }

    function getWeather(){
      api.get(`${ apiUrl.value }?lat=${ latitude.value }&lon=${ longitude.value }&appid=${ apiKey.value }`)
      .then(response => { 
          weatherData.value = response.data})
          .then(response => { 
            console.log('response: ', response)})           
    }

    return {
      location, 
      weatherData,

      getLocation

    }
  }
});

</script>

<style lang="scss">
  .q-page {
    background: linear-gradient(to bottom, #36d1dc, #5b86e5); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  }
  .degree {
    top:-35px
  }
</style>
