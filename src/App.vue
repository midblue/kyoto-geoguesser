<template>
  <div>
    <div class="streetviewcontainer">
      <div id="streetview" ref="streetview"></div>
    </div>
    <div class="right">
      <div class="righttop">
        <div v-if="showActual">
          ここだ！
          <template v-if="distanceOff && distanceOff > 0">
            場所からの距離は
            <span class="highlight">{{ distanceOff.toFixed(2) }}km</span>。
          </template>
        </div>
        <div v-else>どこにいると思いますか？範囲内にクリックして！</div>

        <button v-if="showActual" class="nextbutton" @click="nextLocation">次へ</button>
        <!-- <button
          v-else
          class="nextbutton secondary"
          @click="() => {showActual = true; distanceOff = 0}"
        >諦める</button>-->
      </div>
      <Map
        :actualCoords="{lat, lng}"
        :showActual="showActual"
        @showActual="showActual = true"
        @distanceOff="distance => distanceOff = distance"
      />
    </div>
  </div>
</template>

<script>
import Map from './components/Map'
const randomPointsOnPolygon = require('random-points-on-polygon')

export default {
  components: { Map },
  data() {
    return {
      lat: null,
      lng: null,
      showActual: false,
      distanceOff: null,
      gk: require('../gmapsApiKey.json').key,
      kyotoPolygon: {
        type: 'Feature',
        properties: {},
        geometry: {
          type: 'Polygon',
          coordinates: [
            [
              [135.67480087280273, 35.01425155045957],
              [135.6789207458496, 35.008065256512594],
              [135.69574356079102, 34.99808184104146],
              [135.71428298950195, 34.970234332298475],
              [135.77625274658203, 34.97220365922886],
              [135.79256057739258, 35.04025698454798],
              [135.7774543762207, 35.046159768522124],
              [135.75857162475586, 35.044332761914966],
              [135.73110580444336, 35.02830956921098],
              [135.67480087280273, 35.01425155045957],
            ],
          ],
        },
      },
    }
  },
  watch: {},
  created() {
    let mapsAPIScript = document.createElement('script')
    mapsAPIScript.setAttribute(
      'src',
      `https://maps.googleapis.com/maps/api/js?key=${
        this.gk
      }&callback=initializeGoogleMapsAPI`
    )
    window.initializeGoogleMapsAPI = this.nextLocation
    document.head.appendChild(mapsAPIScript)
  },
  methods: {
    getRandomPoint() {
      return randomPointsOnPolygon(1, this.kyotoPolygon)[0]
    },
    nextLocation() {
      this.showActual = false
      const point = this.getRandomPoint().geometry.coordinates
      const heading = Math.floor(Math.random() * 360)
      // console.log(point[1], point[0], heading)
      this.lat = point[1]
      this.lng = point[0]

      const pano = new google.maps.StreetViewPanorama(this.$refs.streetview, {
        position: { lat: this.lat, lng: this.lng },
        pov: { heading, pitch: 0 },
        // clickToGo: false,
        showRoadLabels: false,
        zoom: 0,
      })
      setTimeout(() => {
        if (!pano.pano) {
          console.log('pano failed!')
          this.nextLocation()
        }
      }, 700)
    },
  },
}
</script>

<style>
html,
body {
  margin: 0;
  font-family: 'Avenir', sans-serif;
}

.streetviewcontainer {
  position: fixed;
  z-index: 3;
  top: 0;
  left: 0;
  height: 100vh;
  width: 50vw;
}
#streetview {
  height: 100vh;
  width: 50vw;
}

.gm-control-active,
.gm-iv-address,
.gm-style-cc,
.gmnoprint,
#streetview img,
.gmnoscreen {
  display: none;
}

.right {
  height: 100vh;
  position: absolute;
  width: 50vw;
  left: 50%;
  top: 0;
  display: grid;
  grid-template-rows: 80px 1fr;
}

.right .righttop {
  background: floralwhite;
  color: #444;
  position: relative;
  text-align: center;
  height: 100%;
  display: flex;
  justify-content: stretch;
  align-items: center;
  font-size: 1.8rem;
}
.right .righttop > * {
  flex-grow: 1;
}
.highlight {
  color: green;
  font-weight: 600;
}

button {
  height: 100%;
  border: none;
  font-size: 1em;
  padding: 0 30px;
  background: green;
  color: white;
  transition: all 0.2s;
  color: white;
  outline: 0;
}

button:hover {
  background: darkgreen;
  cursor: pointer;
}

button.secondary {
  background: gray;
}
</style>
