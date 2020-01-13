<template>
  <div class="bg-indigo-800 h-screen flex flex-col p-8">
    <h1 class="text-white text-5xl font-thin leading-none mb-8">
      Komm ich trocken an?
    </h1>

    <div v-if="showMap" class="-mx-8 maps-container">
      <iframe
        width="600"
        height="450"
        frameborder="0"
        style="border:0"
        :src="`https://www.google.com/maps/embed/v1/directions?origin=${addressFrom.latitude},${addressFrom.longitude}&destination=${addressTo.latitude},${addressTo.longitude}&mode=${type}&key=${mapsKey}`"
        allowfullscreen
      />
    </div>

    <div class="flex flex-row flex-wrap">
      <div class="md:w-6/12 w-full py-2 md:py-0 md:px-2">
        <div v-if="!showBike && !showByFoot" class="bg-purple-700 rounded-lg shadow-lg py-2 px-4 text-white">
          <div class="flex align-middle">
            <div class="font-bold text-2xl tracking-wide">
              Fahrrad
            </div>
            <div class="font-bold text-4xl ml-auto" @click="showBike = true">
              +
            </div>
          </div>
        </div>
        <div v-if="showBike" class="bg-purple-700 rounded-lg shadow-lg py-4 px-4 text-white" :class="{'-mt-8 relative': showMap}">
          <label for="start" class="mb-2">Start</label>
          <vue-google-autocomplete
            id="start"
            ref="byBike"
            classname="w-full h-8 rounded text-black px-2"
            placeholder="Start typing"
            @placechanged="getFromData"
          />
          <label for="stop" class="mb-2">Stop</label>
          <vue-google-autocomplete
            id="stop"
            ref="byBike"
            classname="w-full h-8 rounded text-black px-2"
            placeholder="Start typing"
            @placechanged="getToData"
          />
        </div>
      </div>
      <div class="md:w-6/12 w-full py-2 md:py-0 md:px-2">
        <div v-if="!showByFoot && !showBike" class="bg-purple-700 rounded-lg shadow-lg py-2 px-4 text-white">
          <div class="flex align-middle">
            <div class="font-bold text-2xl tracking-wide">
              Zu Fuss
            </div>
            <div class="font-bold text-4xl ml-auto" @click="showByFoot = true">
              +
            </div>
          </div>
        </div>
        <div v-if="showByFoot" class="bg-purple-700 rounded-lg shadow-lg py-2 px-4 text-white">
          <label for="start" class="mb-2">Start</label>
          <vue-google-autocomplete
            id="start"
            ref="byFoot"
            classname="w-full h-8 rounded text-black px-2"
            placeholder="Start typing"
            @placechanged="getFromData"
          />
          <label for="stop" class="mb-2">Stop</label>
          <vue-google-autocomplete
            id="stop"
            ref="byFoot"
            classname="w-full h-8 rounded text-black px-2"
            placeholder="Start typing"
            @placechanged="getToData"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import Logo from '~/components/Logo.vue'
import VueGoogleAutocomplete from 'vue-google-autocomplete'
import axios from 'axios'
import config from '@/config'

export default {
  components: {
    VueGoogleAutocomplete
  },

  data () {
    return {
      showBike: false,
      showByFoot: false,
      address: '',
      showMap: false,
      weather: null,
      mapsKey: config.maps.secret,
      weatherKey: config.weather.secret
    }
  },

  methods: {
    parseWeather (weather) {
      console.log(weather)

      if (weather.weather[0].main === 'Rain') {
        console.log('rain')
      } else {
        console.log('no rain')
      }
    },

    lookForWeather (lat, lon) {
      axios
        .get(`http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&APPID=${this.weatherKey}&units=metric`)
        .then((response) => {
          this.weather = response.data
        })
        .finally(() => this.parseWeather(this.weather)
        )
    },

    getFromData (addressData, placeResultData, id) {
      this.addressFrom = addressData
      if (this.$refs.byBike) {
        this.type = 'bicycling'
      } else if (this.$refs.byFoot) {
        this.type = 'walking'
      } else {
        console.log('other')
      }
    },

    getToData (addressData, placeResultData, id) {
      this.addressTo = addressData
      this.showMap = true
      // console.log(this.addressTo.latitude, this.addressTo.longitude)
      this.lookForWeather(this.addressTo.latitude, this.addressTo.longitude)
    }
  }
}
</script>

<style>
/* Sample `apply` at-rules with Tailwind CSS
.container {
  @apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/

.maps-container {
  @apply relative;
  padding-bottom: 100%;
  height: 0;
  overflow: hidden;

}

.maps-container iframe {
  @apply absolute inset-0 w-full h-full;
}

</style>
