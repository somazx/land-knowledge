<template>
  <el-container>
    <el-main>
      <h1>Land Knowledge Demo</h1>
      <el-autocomplete
      class="inline-input"
      v-model="address"
      :fetch-suggestions="fetchSuggestions"
      placeholder="Property Address"
    ><i slot="suffix" class="el-input__icon el-icon-location"></i></el-autocomplete>
    
    <el-button type="primary" plain>View Report</el-button>

    </el-main>
  </el-container>
</template>

<script lang="ts">
import Vue from 'vue'
import axios from "axios"

const GEOCODER_API_URL = "https://geocoder.api.gov.bc.ca/addresses.json"

async function geoFetch(addressString:string) {
  const url = GEOCODER_API_URL
  const params = {
    addressString,
    maxResults: 5,
    brief: true,
    autoComplete: true,
    minScore: 50,
    echo: false
  }
  const response = await axios.get(GEOCODER_API_URL, { params })
  
  return response.data
}

function parseGeoFetchSuggestions(responseJson:GeoCoderResponse) {
    return responseJson.features.map(({ geometry, properties }) => ({ 
        value: properties.fullAddress,
        link: geometry.coordinates.join(",")
      })
    )
}


interface GeoResponseProperties {
  fullAddress: string
  [key: string]: any
}

interface GeoResponseGeometry {
  coordinates: Float32Array[]
}

interface GeoResponseFeature {
  geometry: GeoResponseGeometry
  properties: GeoResponseProperties
  [key: string]: any
}

interface GeoCoderResponse {
  features: GeoResponseFeature[]
  [key: string]: any
}

export default Vue.extend({
  name: 'IndexPage',
  data: () => ({
    address: "",
    suggestions: []
  }),
  methods: {
    async fetchSuggestions(queryString: string, cb: (arg0: { value: string; link: string }[]) => void) {
      const responseJson = await geoFetch(queryString)
      
      cb(parseGeoFetchSuggestions(responseJson))
    }
  }
})
</script>

<style>
body, html {
  font-family: Arial, Helvetica, sans-serif;
  background-color:lightslategray;
  margin: 0px
}

body, html, #__nuxt, #__layout, .el-container, .el-main {
  height: 100%;
}

.el-container {
  max-width: 1024px;
  margin: 0 auto;
  background-color: white;
}
</style>