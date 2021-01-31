<template>
  <h1>Vue 3 / Mapbox - Single File Component</h1>
  <div id="map" />
  <button id="test-reactivity" @click="title = 'Popup info changed!'">Test Reactivity - change popup title!</button>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";
import { onMounted, createApp, defineComponent, ref, nextTick } from "vue";
import popupContent from "@/components/popupContent.vue";

export default {
  setup() {
    const title = ref("EU Border Region");
    onMounted(() => {
      mapboxgl.accessToken = "yourAccessToken";
      const map = new mapboxgl.Map({
        container: "map",
        center: [22, 56],
        zoom: 2,
        style: "mapbox://styles/mapbox/light-v9"
      });
      map.on('load', () => {
        // Loading a layer
        map.addSource("eu", {
          type: "geojson",
          data: "https://raw.githubusercontent.com/leakyMirror/map-of-europe/27a335110674ae5b01a84d3501b227e661beea2b/GeoJSON/europe.geojson"
        });
        map.addLayer({
          id: "eu-fill",
          type: "fill",
          source: "eu",
          paint: {
            "fill-color": "#2C5364",
          },
        });
        // Setting up the dropdown
        map.on("click", "eu-fill", function(e) {
          new mapboxgl.Popup()
            .setLngLat(e.lngLat)
            .setHTML('<div id="popup-content"></div>')
            .addTo(map);
          const newPopup = defineComponent({
            extends: popupContent,
            setup() {
              // const title = 'EU border region'
              return { title };
            },
          });
          nextTick(() => {
            createApp(newPopup).mount('#popup-content');
          });
        });
      });
    });
    return { title };
  },
};

</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;800&display=swap');

#app {
  font-family: 'Montserrat', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

#map {
  height: 50vh;
  width: 50vw;
  border: 2px solid #2C5364;
  border-radius: 10px;
}

button#test-reactivity {
  box-sizing: border-box;
  appearance: none;
  background-color: transparent;
  border: 2px solid #e74c3c;
  border-radius: 0.6em;
  color: #e74c3c;
  cursor: pointer;
  display: flex;
  align-self: center;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1;
  margin: 20px;
  padding: 1.2em 2.8em;
  text-decoration: none;
  text-align: center;
  text-transform: uppercase;
  font-family: 'Montserrat', sans-serif;
  font-weight: 700;
  transition: box-shadow 300ms ease-in-out, color 300ms ease-in-out;
}

button#test-reactivity:hover {
  color: #fff;
  outline: 0;
  box-shadow: 0 0 40px 40px #e74c3c inset;
}

div.mapboxgl-popup-content {
  font-family: 'Montserrat', sans-serif;
  padding-top: 22px;
}

button.mapboxgl-popup-close-button {
  font-size: 18px;
}

</style>
