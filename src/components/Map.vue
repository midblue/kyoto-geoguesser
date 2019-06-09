<template>
  <MapboxMapLoader
    :apikey="ak"
    mapboxStyle="mapbox://styles/midblue/cjv2c454j6eoy1fnx66ks9tpw"
    :actualCoords="actualCoords"
    :showActual="showActual"
    @showActual="$emit('showActual')"
    @distanceOff="distance => $emit('distanceOff', distance)"
  >
    <template slot-scope="{ mapboxgl, map, clusterer }"></template>
  </MapboxMapLoader>
</template>

<script>
//mapbox://styles/mariko9012/cjh4gkzlw31mc2sqsm3l0g4rk
import MapboxMapLoader from './MapboxMapLoader'

export default {
  props: ['actualCoords', 'showActual'],
  components: {
    MapboxMapLoader,
  },
  data() {
    return {
      ak: require('../../mapboxApiKey.json').key,
      markers: [],
      updatePrefix: 0,
    }
  },

  computed: {},

  watch: {},

  methods: {},
}
</script>

<style></style>
