<template>
  <div>
    <fieldset>
      <p>Center: {{ center }}, Zoom: {{ zoom }}</p>
      <button @click="createMarkers" :disabled="markers.length > 0">Create markers</button>
      <button @click="removeMarkers" :disabled="markers.length === 0">Delete markers</button>
      <button @click="test" :disabled="markers.length > 0">Stress-test marker generation</button>
    </fieldset>
    <div ref="map" class="map"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { EventEmitter } from 'events'
import L, { LatLng } from 'leaflet'
import data from './random-coordinates.json'
import voivodeships from './wojewodztwa-min.geo.json'
import counties from './powiaty-min.geo.json'
import { GeoJsonObject } from 'geojson'

type Nullable<T> = T | null

const HOME = {
  lat: 50.32269481418845,
  lng: 18.656292557716373,
  // lat: 46.8,
  // lng: 8.1,
}

const MAX_ZOOM = 19

const OSM_ATTRIB = '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
const OSM_URL = 'https://tile.openstreetmap.org/{z}/{x}/{y}.png'
const MB_ATTRIB = 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>'
const MB_URL = 'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw'

const ICON_ON = L.icon({ iconUrl: '/default-marker-highlighted.png' })
const ICON_OFF = L.icon({ iconUrl: '/default-marker.png' })

class MyMarker extends EventEmitter {
  public readonly data: L.Marker

  constructor(
    public readonly map: L.Map,
    public readonly id: string,
    coords: LatLng,
  ) {
    super()
    this.data = L
      .marker(coords)
      .setIcon(ICON_OFF)
      .on('click', e => {
        this.click()
      })
      .on('mouseover', e => {
        this.highlight()
      })
      .on('mouseout', e => {
        this.unhighlight()
      })
      .addTo(this.map)
  }

  remove() {
    this.removeAllListeners()
    this.data.clearAllEventListeners
    this.data.remove()
  }

  click() {
    this.emit('click', this)
    console.log('marker clicked', this)
  }

  highlight() {
    this.data.setIcon(ICON_ON)
    console.log('marker hovered', this)
  }

  unhighlight() {
    this.data.setIcon(ICON_OFF)
    console.log('marker left', this)
  }
}

export default defineComponent({
  data() {
    return {
      map: null as Nullable<L.Map>,
      markers: [] as MyMarker[],
      center: HOME,
      zoom: MAX_ZOOM
    }
  },
  mounted() {
    const map = L.map(this.$refs.map as HTMLElement).setView(HOME, 9)
    const osm = L.tileLayer(OSM_URL, {
      maxZoom: MAX_ZOOM,
      attribution: OSM_ATTRIB
    })
    const satellite = L.tileLayer(MB_URL, {
      id: 'mapbox/satellite-v9',
      tileSize: 512,
      zoomOffset: -1,
      attribution: MB_ATTRIB
    })
    // @ts-ignore
    const voivodeship = L.geoJSON(voivodeships as GeoJsonObject, { interactive: true })
      .on('mouseover', (e) => {
        voivodeship.setStyle({ opacity: 0.5 })
      })
      .on('mouseout', (e) => {
        voivodeship.setStyle({ opacity: 0.8 })
      })
      .on('add', e => console.log('add', e.target))
    // voivodeship.
    // const county = L.geoJSON(counties)



    map
      // .addLayer(osm)
      .addLayer(satellite)
      .addLayer(voivodeship)
      // .addLayer(L.marker(this.center).on('click', () => { map.setView(HOME, MAX_ZOOM) }))
      .on('zoom', () => this.zoom = map.getZoom())
      .on('move', () => this.center = map.getCenter())


    // @ts-ignore
    window.L = L
    // @ts-ignore
    window.map = map
    this.map = map
    this.center = map.getCenter()
  },
  beforeUnmount() {
    this.map!.remove()
    this.map = null
  },
  methods: {
    async test() {
      for (let i = 0; i < 10; i++) {
        this.createMarkers()
        await this.$nextTick()
        this.removeMarkers()
        await this.$nextTick()
      }
    },
    createMarkers() {
      console.time('Adding markers')
      data.forEach(point => {
        const marker = new MyMarker(this.map as any, point.id, L.latLng({ lat: point.lat, lng: point.lon }))
        marker.on('click', () => { console.log('marker clicked') })
        this.markers.push(marker)
      })
      console.timeEnd('Adding markers')
    },
    removeMarkers() {
      console.time('Removing markers')
      this.markers.forEach(marker => { marker.remove() })
      this.markers = []
      console.timeEnd('Removing markers')
    }
  }
})
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

.map {
  height: 100vh;
}
</style>
