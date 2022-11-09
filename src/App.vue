<template>
  <div>
    <fieldset>
      <p>Center: {{ center }}, Zoom: {{ zoom }}</p>
    </fieldset>
    <div ref="map" class="map"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { EventEmitter } from 'events'
import L, { LatLng } from 'leaflet'
import data from './random-coordinates.json'
import counties from './powiaty-min.geo.json'
import { GeoJsonObject } from 'geojson'
import { v4 as uuid } from 'uuid'
import 'leaflet.markercluster/dist/MarkerCluster.css'
import 'leaflet.markercluster/dist/MarkerCluster.Default.css'
import 'leaflet.markercluster'

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

export default defineComponent({
  data() {
    return {
      map: null as Nullable<L.Map>,
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

    const NUMBER_OF_MARKERS = 50000

    console.time('Creating ' + NUMBER_OF_MARKERS + ' random marker points')
    const markers = new Array(NUMBER_OF_MARKERS).fill({ lat: 0, lon: 0, id: 0 }).map(marker => {
      const r = 1 * Math.sqrt(Math.random())
      const theta = Math.random() * 2 * Math.PI

      return {
        id: uuid(),
        lat: HOME.lat + r * Math.cos(theta),
        lon: HOME.lng + r * Math.sin(theta) * 2,
      }
    })
    console.timeEnd('Creating ' + NUMBER_OF_MARKERS + ' random marker points')

    console.time('Creating cluster')
    const clusters = L.markerClusterGroup()
    const toMarker = (marker: any) => L
      .marker([ marker.lat, marker.lon ], { icon: ICON_OFF })
      .on('mouseover', e => { e.target.setIcon(ICON_ON) })
      .on('mouseout', e => { e.target.setIcon(ICON_OFF) })

    markers
      .map(toMarker)
      .forEach(marker => {
        clusters.addLayer(marker)
      })
    console.timeEnd('Creating cluster')

    console.time('Creating map')
    map
      .addLayer(osm)
      // .addLayer(satellite)
      .addLayer(clusters)
      // .addLayer(L.marker(this.center).on('click', () => { map.setView(HOME, MAX_ZOOM) }))
      .on('zoom', () => this.zoom = map.getZoom())
      .on('move', () => this.center = map.getCenter())

    console.timeEnd('Creating map')

    // clusters.addTo(map)

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
    createMarkers() {
    },
    removeMarkers() {
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
