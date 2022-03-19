<template>
  <div class="flex flex-col h-screen max-h-screen">
    <div
      class="
        z-20
        flex
        justify-center
        relative
        px-4
        pt-8
        pb-32
        bg-hero-pattern bg-cover
      "
    >
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input
            v-model="queryIp"
            type="text"
            name=""
            class="
              flex-1
              py-3
              px-2
              rounded-tl-md rounded-bl-md
              focus:outline-none
            "
            placeholder="Search for any IP address or leave empty to get your ip info"
          />
          <i
            @click="getIpInfo"
            class="
              cursor-pointer
              bg-black
              text-white
              px-4
              rounded-tr-md rounded-br-md
              flex
              items-center
              fas
              fa-chevron-right
            "
          ></i>
        </div>
      </div>
      <IpInfo v-if="ipInfo" :ipInfo="ipInfo"></IpInfo>
    </div>

    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
import IpInfo from "../components/IpInfo.vue";
import leaflet from "leaflet";
import { onMounted, ref } from "vue";
import axios from "axios";

export default {
  name: "HomeView",
  components: { IpInfo },
  setup() {
    let map;

    const queryIp = ref("");
    const ipInfo = ref(null);

    onMounted(() => {
      map = leaflet.map("map").setView([51.505, -0.09], 13);

      leaflet
        .tileLayer(
          "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken: "mapbox_token",
          }
        )
        .addTo(map);
    });

    const getIpInfo = async () => {
      const apiKey = "ipify_token";
      const response = await axios.get(
        `https://geo.ipify.org/api/v1/?apiKey=${apiKey}&ipAddress=${queryIp.value}`
      );

      const result = response.data;
      ipInfo.value = {
        address: result.ip,
        state: result.location.region,
        timezone: result.location.timezone,
        isp: result.isp,
        lat: result.location.lat,
        lng: result.location.lng,
      };
      leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(map);
      map.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
    };

    return { queryIp, ipInfo, getIpInfo };
  },
};
</script>
