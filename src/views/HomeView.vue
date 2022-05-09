<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- search -->
    <div class="z-20 relative flex justify-center bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- search input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address</h1>
        <div class="flex ">
          <input v-model="queryIp" type="text" class="flex-1 py-3 px-2 rounded-tl-md focus:outline-none" placeholder="Search for any IP address or leave empty to get your ip info">
          <i @click="getIpInfo" class="flex items-center cursor-pointer bg-pink-500 text-white px-4 rounded-tr-md rounded-br-md fas fa-chevron-right"></i>
        </div>
      </div>
      <!-- IP info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo"/>
    </div>
    <!-- map -->
    <div id="mapid" class="h-full z-10" ></div>
  </div>
</template>

<script>
// @ is an alias to /src
import IPInfo from "../components/IPInfo.vue"
import leaflet from "leaflet"
import axios from "axios"
import { onMounted, ref } from "vue"

export default {
  name: 'HomeView',
  components: {
    IPInfo
  },
  setup() {
    let mymap
    const queryIp = ref("")
    const ipInfo = ref(null)
    onMounted(() => {
      mymap = leaflet.map('mapid').setView([11.50709191201587, 107.32800387002484], 7);
      leaflet.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1,
          accessToken: 'pk.eyJ1IjoianVuZ3Nvb2ppbiIsImEiOiJjbDJ5dDQwbGwxMm9uM2NsZ2w4a2lvcHF5In0.-LPO4qiiknTV8z-oYCSlDw'
      }).addTo(mymap)
    })
    

    const getIpInfo = async () => {
      try {
        const data = await axios.get(`https://geo.ipify.org/api/v2/country,city?apiKey=at_w4wqVhdOubO6WuVtAu5mP33Mj7iZs&ipAddress=${queryIp.value}`)
        const result = data.data
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        }
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13)
      }
      catch(err) {
        console.log(err.message)
      }
    }
    return {queryIp, ipInfo, getIpInfo}
  }
}
</script>
