<template>
  <l-map
    class="leaflet-container"
    :center="[
      /* Centre of Sydney from OpenStreetMap https://www.openstreetmap.org/node/13766899 */
      -33.8698439,
      151.2082848,
    ]"
    :zoom="14"
    ref="map"
  >
    <l-tile-layer
      url="https://stamen-tiles-{s}.a.ssl.fastly.net/toner-lite/{z}/{x}/{y}{r}.png"
      layer-type="base"
      name="Minimal map style"
      :min-zoom="0"
      :max-zoom="18"
      attribution="Map tiles by <a href='http://stamen.com'>Stamen Design</a>, <a href='http://creativecommons.org/licenses/by/3.0'>CC BY 3.0</a> &mdash; Map data &copy; <a href='https://www.openstreetmap.org/copyright'>OpenStreetMap</a> contributors"
      subdomains="abcd"
      :opacity="0.8"
    ></l-tile-layer>
    <l-tile-layer
      url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      layer-type="base"
      name="Detailed map style"
      :max-zoom="19"
      attribution="&copy; <a href='https://www.openstreetmap.org/copyright'>OpenStreetMap</a> contributors"
      :visible="false"
    ></l-tile-layer>
    <l-tile-layer
      url="https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png"
      layer-type="base"
      name="Cycling map style"
      :max-zoom="20"
      attribution="<a href='https://github.com/cyclosm/cyclosm-cartocss-style/releases' title='CyclOSM - Open Bicycle render'>CyclOSM</a> | Map data: &copy; <a href='https://www.openstreetmap.org/copyright'>OpenStreetMap</a> contributors"
      :visible="false"
    ></l-tile-layer>
    <l-tile-layer
      url="https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm-lite/{z}/{x}/{y}.png"
      layer-type="overlay"
      name="Show cycle infrastructure"
      :max-zoom="20"
      attribution="<a href='https://github.com/cyclosm/cyclosm-cartocss-style/releases' title='CyclOSM - Open Bicycle render'>CyclOSM</a> | Map data: &copy; <a href='https://www.openstreetmap.org/copyright'>OpenStreetMap</a> contributors"
      :visible="false"
    ></l-tile-layer>
    <l-control-layers />
    <l-circle-marker
      v-for="crash of crashes"
      :key="crash.id"
      :lat-lng="[crash.lat, crash.lon]"
      :radius="$store.state.currentCrash?.id === crash.id ? 14 : 8"
      @click="handleClick"
      :color="getColorForDegree(crash.deg)"
      :fill-color="getColorForDegree(crash.deg)"
      :fill="true"
      :fill-opacity="0.8"
    />
    <l-polyline
      v-if="$store.state.routeCoords"
      :lat-lngs="$store.state.routeCoords"
      color="#4bc34b"
      ref="route"
      @ready="handleRouteReady"
    />
  </l-map>
</template>

<script>
import crashes from "../data/built/crashes.json";
import "leaflet/dist/leaflet.css";
import {
  LMap,
  LTileLayer,
  LCircleMarker,
  LControlLayers,
  LMarker,
  LPolyline,
} from "@vue-leaflet/vue-leaflet";

export default {
  components: {
    LMap,
    LTileLayer,
    LCircleMarker,
    LControlLayers,
    LMarker,
    LPolyline,
  },
  data() {
    return { crashes };
  },
  methods: {
    handleClick(e) {
      // Find crash that matches coords of marker clicked
      const crash = crashes.find(
        (c) => c.lat === e.latlng.lat && c.lon === e.latlng.lng
      );
      this.$store.commit("setCurrentCrash", crash);
    },
    handleRouteReady() {
      if (this.$refs.route && this.$refs.map) {
        const bounds = this.$refs.route.leafletObject.getBounds();
        const map = this.$refs.map.leafletObject;
        map.flyToBounds(bounds);
      }
    },
    getColorForDegree(degree) {
      switch (degree) {
        case "Non-casualty (towaway)":
          return "#868e96"; // Gray
        case "Minor/Other Injury":
          return "#339af0"; // Blue
        case "Moderate Injury":
          return "#fab005"; // Yellow
        case "Serious Injury":
          return "#f76707"; // Orange
        case "Fatal":
          return "#c92a2a"; // Red
        default:
          return "blue"; // Eye-burn blue so I notice something's wrong hopefully
      }
    },
  },
};
</script>

<style lang="scss">
.leaflet-container {
  background: transparent;
}
</style>
