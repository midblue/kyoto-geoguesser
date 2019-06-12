<template>
  <div style="height: 100%;">
    <div id="map" :class="{ ready: styleReady }"></div>
    <template v-if="styleReady">
      <slot :mapboxgl="mapboxgl" :map="map"/>
    </template>
  </div>
</template>

<script>
const circleToPolygon = require('circle-to-polygon')

export default {
  props: ['apikey', 'mapboxStyle', 'actualCoords', 'showActual'],

  data() {
    return {
      mapboxgl: null,
      map: null,
      styleReady: false,
      actualMarker: null,
      userCoords: null,
      userMarker: null,
      startTime: Date.now(),
      polygon: [
        // [135.73848724365234, 35.07946034047981],
        // [135.72509765625, 35.03983534080149],
        // [135.66020965576172, 35.028028432524266],
        // [135.68252563476562, 34.9909099035313],
        // [135.65196990966797, 34.9869721019795],
        // [135.6711959838867, 34.91549624729326],
        // [135.72784423828125, 34.906768549797434],
        // [135.77178955078125, 34.93097858831627],
        // [135.78689575195312, 34.95799531086792],
        // [135.7958221435547, 35.016500995886005],
        // [135.7965087890625, 35.06653514925735],
        // [135.76904296875, 35.07271701786369],
        // [135.76148986816406, 35.062600989085496],
        // [135.73848724365234, 35.07946034047981],
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
      defaultPosition: {
        bearing: 0,
        center: [135.748679, 35.0],
        zoom: 10.2,
        pitch: 0,
        speed: 2,
      },
    }
  },

  watch: {
    actualCoords() {
      if (this.userMarker && !this.showActual) {
        this.startTime = Date.now()
        this.userMarker.remove()
      }
    },
    showActual(shouldShow) {
      if (!this.mapboxgl || !this.map) return
      if (this.actualMarker || !shouldShow) this.actualMarker.remove()

      this.fitToKyoto()

      if (shouldShow)
        this.actualMarker = new this.mapboxgl.Marker({ color: '#ffbb00' })
          .setLngLat(this.actualCoords)
          .addTo(this.map)
      else {
        if (this.map && this.mapboxgl && this.map.getLayer('linebetween')) {
          this.map.removeLayer('linebetween')
          this.map.removeSource('linebetween')
          this.map.removeLayer('circlearound')
          this.map.removeSource('circlearound')
        }
      }
    },
    userCoords(newCoords) {
      if (!this.mapboxgl || !this.map) return
      if (this.userMarker || !newCoords) this.userMarker.remove()

      this.fitToKyoto()

      if (newCoords) {
        this.map.addLayer({
          id: 'linebetween',
          type: 'line',
          source: {
            type: 'geojson',
            data: {
              type: 'Feature',
              geometry: {
                type: 'LineString',
                coordinates: [
                  [this.actualCoords.lng, this.actualCoords.lat],
                  newCoords,
                ],
              },
            },
          },
          layout: {},
          paint: {
            'line-color': '#77bbff',
            'line-width': 2,
          },
        })

        const polygon = circleToPolygon(
          [this.actualCoords.lng, this.actualCoords.lat],
          1000,
          32
        )

        this.map.addLayer({
          id: 'circlearound',
          type: 'fill',
          source: {
            type: 'geojson',
            data: {
              type: 'Feature',
              geometry: polygon,
            },
          },
          layout: {},
          paint: {
            'fill-color': 'rgba(255,187,0,.3)',
          },
        })

        this.userMarker = new this.mapboxgl.Marker({ color: '#77bbff' })
          .setLngLat(newCoords)
          .addTo(this.map)

        const distance = getDistanceFromLatLonInKm(
          this.actualCoords.lng,
          this.actualCoords.lat,
          ...newCoords
        )
        this.$emit('distanceOff', distance)

        const payloadToDatabase = {
          actualCoords: [this.actualCoords.lng, this.actualCoords.lat],
          userCoords: newCoords,
          date: new Date(),
          distanceOffInKm: distance,
          timeTakenInMs: Date.now() - this.startTime,
        }
        fetch('http://0.0.0.0:3008/dataupload', {
          method: 'post',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payloadToDatabase),
        })
          .then(res => {
            return res.json()
          })
          .then(data => {
            if (!data || data.status !== 'ok')
              console.error('Failed on server!', data.status)
            else console.info('saved!')
          })
      }
    },
  },

  mounted() {
    if (!document) return
    this.mapboxgl = require('mapbox-gl/dist/mapbox-gl.js')
    this.mapboxgl.accessToken = this.apikey

    this.map = new this.mapboxgl.Map({
      container: 'map',
      style: this.mapboxStyle,
      ...this.defaultPosition,
    })

    this.map.on('styledata', () => {
      setTimeout(() => (this.styleReady = true), 300)

      this.map.setLayoutProperty('country-label', 'text-field', [
        'get',
        'name_ja',
      ])
      this.map.setLayoutProperty('state-label', 'text-field', [
        'get',
        'name_ja',
      ])
      this.map.setLayoutProperty('place-city-label', 'text-field', [
        'get',
        'name_ja',
      ])

      this.map.setLayoutProperty(
        'place-town-village-hamlet-label',
        'text-field',
        ['get', 'name_ja']
      )

      this.map.setLayoutProperty(
        'place-neighborhood-suburb-label',
        'text-field',
        ['get', 'name_ja']
      )

      this.map.setLayoutProperty('road-label', 'text-field', ['get', 'name_ja'])
    })

    this.map.on('load', () => {
      this.fitToKyoto()
      this.map.addLayer({
        id: 'kyotooutlinelayer',
        type: 'line',
        source: {
          type: 'geojson',
          data: {
            type: 'Feature',
            geometry: {
              type: 'Polygon',
              coordinates: [this.polygon],
            },
          },
        },
        layout: {},
        paint: {
          'line-color': '#0099ff',
          'line-width': 3,
        },
      })
    })

    this.map.on('click', e => {
      if (!e.lngLat) return
      if (this.showActual) {
        return
      }
      this.$emit('showActual')
      this.userCoords = [e.lngLat.lng, e.lngLat.lat]
    })
  },

  methods: {
    fitToKyoto() {
      this.map.fitBounds(
        [
          [
            Math.max(...this.polygon.map(p => p[0])),
            Math.max(...this.polygon.map(p => p[1])),
          ],
          [
            Math.min(...this.polygon.map(p => p[0])),
            Math.min(...this.polygon.map(p => p[1])),
          ],
        ],
        { padding: { top: 20, left: 20, bottom: 20, right: 20 } }
      )
    },
  },
}

function getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2) {
  var R = 6371 // Radius of the earth in km
  var dLat = deg2rad(lat2 - lat1) // deg2rad below
  var dLon = deg2rad(lon2 - lon1)
  var a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(deg2rad(lat1)) *
      Math.cos(deg2rad(lat2)) *
      Math.sin(dLon / 2) *
      Math.sin(dLon / 2)
  var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
  var d = R * c // Distance in km
  return d
}
function deg2rad(deg) {
  return deg * (Math.PI / 180)
}
</script>

<style>
#map {
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 3s;
}

#map,
canvas {
  outline: none !important;
}

#map.ready {
  opacity: 1;
}

#map.focus {
  outline: 0;
}

.mapbox-ctrl-attrib,
.mapboxgl-missing-css,
.mapboxgl-control-container {
  display: none;
}

.mapboxgl-marker {
  position: absolute;
  z-index: 2;
}
</style>
