<template>
  <div id="app">
    <div class="wrapper">
      <toolbox
        :get-data="data"
        :init-select="select"
        @change="updateSelect"
      >
      </toolbox>
      <div class="map-wrapper">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet'
import toolbox from '@/components/toolbox.vue'

let osmMap = {}
const iconsConfig = {
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41]
}
const icons = {
  green: new L.Icon({
    iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
    ...iconsConfig
  }),
  grey: new L.Icon({
    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-grey.png',
    ...iconsConfig
  })
}
const osm = {
  addMapMarker (x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey
    L.marker([y, x], {
      icon
    }).addTo(osmMap).bindPopup(createPop(item))
  },
  removeMapMarker () {
    osmMap.eachLayer((layer) => {
      if (layer instanceof L.Marker) {
        osmMap.removeLayer(layer)
      }
    })
  },
  penTo (x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey
    osmMap.panTo(new L.LatLng(y, x))
    L.marker([y, x], {
      icon
    }).addTo(osmMap).bindPopup(createPop(item)).openPopup()
  }
}
const initialPop = '<div class="initialPop">Your Location</div>'
const createPop = (pharmacy) => (`
  <div class="list-item">
    <div class="name"><h4>${pharmacy.name}</h4></h4><span class="distance">10m</span></div>
    <div class="info"><div class="item">${pharmacy.updated}</div></div>
    <div class="mask-num">
      <div class="round">成人口罩<label>${pharmacy.mask_adult}</label></div>
      <div class="round">兒童口罩<label>${pharmacy.mask_child}</label></div>
    </div>
    <div class="action-group">
      <a class="favorite" href="#"></a>
      <a class="location" href="https://www.google.com.tw/maps/place/${pharmacy.address}" target="_blank" title="Google Map"></a>
    </div>
  </div>
`)
export default {
  name: 'app',
  components: {
    toolbox
  },
  data: () => ({
    data: [],
    osmMap: {},
    select: {
      city: '',
      area: ''
    },
    errMsg: null,
    gettingLocation: false,
    currenLoaction: null
  }),
  created () {
    if (!('geolocation' in navigator)) {
      this.errMsg = 'Geolocation is not available.'
    }
    this.gettingLocation = true
    navigator.geolocation.getCurrentPosition(pos => {
      this.gettingLocation = false
      this.currenLoaction = [pos.coords.latitude, pos.coords.longitude]
      osmMap = L.map('map', {
        center: this.currenLoaction,
        zoom: 15
      })
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>'
      }).addTo(osmMap)
      L.marker(this.currenLoaction).addTo(osmMap).bindPopup(initialPop).openPopup()
    }, err => {
      this.gettingLocation = true
      this.errMsg = err.message
    })
  },
  mounted () {
    this.$http.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
      .then((response) => {
        this.data = response.data.features
      })
  },
  methods: {
    updateMarker () {
      const pharmacies = this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.county === this.select.city
        }
        return pharmacy.properties.town === this.select.area
      })
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy
        osm.addMapMarker(
          geometry.coordinates[0],
          geometry.coordinates[1],
          properties
        )
      })
      this.penTo(pharmacies[0])
    },
    removeMarker () {
      osm.removeMapMarker()
    },
    penTo (pharmacy) {
      const { properties, geometry } = pharmacy
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties)
    },
    updateSelect () {
      this.removeMarker()
      this.updateMarker()
      const pharmacy = this.data.find(item => item.properties.town === this.select.area)
      const { geometry, properties } = pharmacy
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties)
    }
  }
}
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap.scss";
@import "@/assets/style/main.scss";

* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.wrapper {
  display: flex;
}
#map {
  overflow: hidden;
  height: 100vh;
}
</style>
