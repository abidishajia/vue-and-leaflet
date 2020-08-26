<template>
  <div id="container">
    <div id="info">
      <div v-if="this.attraction.name">
        <p>
          <strong>Name:</strong>
          {{this.attraction.name}}
        </p>
        <p>
          <strong>Address:</strong>
          {{this.attraction.address}}
        </p>
        <p>
          <strong>Type:</strong>
          {{this.attraction.category}}
        </p>
      </div>
    </div>
    <div id="mapContainer"></div>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import data from "./Historic-Landmarks.json";
import axios from "axios";

export default {
  name: "Map",
  data() {
    return {
      center: [37.7749, -122.4194],
      data: [],
      map: null,
      attraction: {
        name: "",
        address: "",
        category: "",
      },
      clientSecret: "FTZGMLOIQWFY3A0ELEZIZSUU3M4EKOJKEPXKWUWTMWK1EY4H",
      clientID: "GOSFGAOZKCSLMWADY1ORYJV2A4GUNNHAHBVWY500S1IM42CS",
    };
  },
  methods: {
    setupLeafletMap () {
      this.map = L.map("mapContainer").setView(this.center, 13);
      L.tileLayer(
        "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
        {
          attribution:
            'Map data © <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: "mapbox/streets-v11",
          accessToken:
            "pk.eyJ1IjoiYWJpZGlzaGFqaWEiLCJhIjoiY2l3aDFiMG96MDB4eDJva2l6czN3MDN0ZSJ9.p9SUzPUBrCbH7RQLZ4W4lQ",
        }
      ).addTo(this.map);
      L.geoJSON(data , {
        onEachFeature: this.onEachFeature,
        style: this.styleMap,
      })
      .addTo(this.map)
      .on("click", this.onClick);
    },
    styleMap(feature){
      const year = feature.properties.datelisted
            ? parseInt(feature.properties.datelisted.slice(0, 4))
            : 0;
          const color = year > 2000 ? "red" : "blue";
          return { color: color };
    },
    onEachFeature(feature, layer) {
      if (feature.properties && feature.properties.name) {
        layer.bindPopup(feature.properties.name);
        layer.on('mouseover', () => { layer.openPopup(); });
        layer.on('mouseout', () => { layer.closePopup(); });
      }
    },
    onClick(e) {
      const name = e.layer.feature.properties.name;
      axios
        .get(
          `https://api.foursquare.com/v2/venues/search?client_id=${this.clientID}&client_secret=${this.clientSecret}&v=20180323&limit=1&near=San Francisco, CA&query=${name}`
        )
        .then((data) => {
          this.attraction = {
            name: name,
            address: data.data.response.venues[0].location.formattedAddress.join(
              " "
            ),
            category: data.data.response.venues[0].categories[0].name,
          };
        });
    },
  },
  mounted() {
    this.setupLeafletMap();
  },
};
</script>

<style scoped>
#container {
  display: flex;
}
#info {
  width: 18vw;
  height: 100vh;
}
#mapContainer {
  width: 80vw;
  height: 100vh;
}
</style>