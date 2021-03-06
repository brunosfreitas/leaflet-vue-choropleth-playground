<template>
  <div id="app">
    <l-map
      v-bind="l_map_props"
      @update:zoom="zoomUpdated"
      @update:center="centerUpdated"
      @update:bounds="boundsUpdated"
    >
      <l-tile-layer v-bind="l_tile_layer_props"></l-tile-layer>
      <l-marker
        v-for="a_marker in l_marker_props"
        :key="a_marker.ley"
        :lat-lng="a_marker.value"
        :name="a_marker.options.title"
        :options="a_marker.options"
      >
        <l-icon v-bind="a_marker.icon"></l-icon>
        <l-popup>{{ a_marker.options.title }}</l-popup>
      </l-marker>

      <l-choropleth-layer
        v-bind="l_choropleth_layer_props"
        @l-add="enteredAnArea"
        @l-click="enteredAnArea"
        @click="enteredAnArea"
      >
        <template slot-scope="props">
          <l-info-control
            :item="props.currentItem"
            :unit="props.unit"
            title="Country info"
            placeholder="Hover over a department"
          />
          <l-reference-chart
            title="Girls school enrolment"
            v-bind="l_reference_chart_props"
            :min="props.min"
            :max="props.max"
          />
        </template>
      </l-choropleth-layer>
    </l-map>
  </div>
</template>

<script>
// As in: https://leafletjs.com/examples/choropleth/
// and: https://github.com/voluntadpear/vue-choropleth
import { InfoControl, ReferenceChart, ChoroplethLayer } from "vue-choropleth";
// import { geojson } from "./data/py-departments-data";

// Data get at: https://github.com/johan/world.geo.json/blob/master/countries.geo.json?short_path=afdfc39
// If you need to import map data from a external server: https://korigan.github.io/Vue2Leaflet/#/components/l-geo-json/
import countriesGeojson from "./data/countries.json";
// import { pyDepartmentsData } from "./data/py-departments-data";
import { countriesData } from "./data/countries-data";
import { LMap, LTileLayer, LMarker, LPopup, LIcon } from "vue2-leaflet";
export default {
  name: "app",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LIcon,
    "l-info-control": InfoControl,
    "l-reference-chart": ReferenceChart,
    "l-choropleth-layer": ChoroplethLayer
  },
  data() {
    return {
      /*
       ** Props for LMap
       */
      l_map_props: {
        // Initial zoom -- From 0-18
        zoom: 5,

        // Jump to the world where the layers exists.
        // TODO Solve a way to only render the map area, without copies - mapBounds
        worldCopyJump: true,

        options: {
          // attributionControl: false,
          minZoom: 3,
          maxZoom: 7
        },

        // Where the map starts - could be the user location
        center: L.latLng(47.41322, -1.219482)
      },

      // A list of markers
      l_marker_props: {
        marker: {
          key: 0,
          // Paris
          value: L.latLng(48.8566, 2.3522),
          options: {
            title: "I'm a point",
            alt: "I'm a point",
            riseOnHover: true
          },
          // More details - https://www.youtube.com/watch?v=JOg5GDy2Ih4&list=PLDmvslp_VR0xjh7wGMNd_1kk0zUox6Sue&index=2
          icon: {
            iconSize: [32, 32],
            // 0,0 is the first pixel of the image, so we want Middle X, bottom 32
            iconAnchor: [16, 32],
            // As its initial point is on the iconAnchor
            popupAnchor: [0, -32],
            className: "c-icon--paris",
            iconUrl: "src/assets/hermes.png"
          }
        },
        marker2: {
          key: 1,
          // London
          value: L.latLng(51.509865, -0.118092),
          options: {
            title: "And the other one",
            opacity: 0.3
          },
          icon: {
            iconSize: [32, 32],
            // 0,0 is the first pixel of the image, so we want Middle X, bottom 32
            iconAnchor: [16, 32],
            // As its initial point is on the iconAnchor
            popupAnchor: [0, -32],
            className: "c-icon--london",
            iconUrl: "src/assets/syringe.png"
          }
        }
      },

      /*
       ** Props for l-choropleth-layer
       */
      l_choropleth_layer_props: {
        // The polygonal paths for countries
        geojson: countriesGeojson,
        // data: Data object with the information to show on the map
        data: countriesData,

        //titleKey: Property of the data object to show when you hover over a certain region of your map (e.g. state_name)
        titleKey: "name",
        //geojsonIdKey: Property under the properties array of the GeoJSON that serves as identifier of each region of the map.
        geojsonIdKey: "id",
        //idKey: Property of the data object that matches the geojsonIdKey value.
        idKey: "id",

        colorScale: ["d7191c", "fdae61", "ffffbf", "a6d96a", "1a9641"],

        // Remove the white stroke of unselected countries
        strokeWidth: 0,
        // strokeColor: "ddff00",
        // Selected/Mouseover country option
        currentStrokeWidth: 2,
        currentStrokeColor: "aaaaaa",
        value: {
          key: "amount_w",
          metric: "% girls"
        },
        extraValues: [
          {
            key: "amount_m",
            metric: "% boys"
          }
        ]
      },

      /*
       ** Props for LTileLayer
       */
      l_tile_layer_props: {
        // -- Colored map with sea borders, local language based names --
        // url: "http://{s}.tile.osm.org/{z}/{x}/{y}.png",

        // -- Grayscale map, english
        url:
          "https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png",
        attribution:
          '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors, &copy; <a href="https://carto.com/attribution" alt="Map tiles by Carto, under CC BY 3.0. Data by OpenStreetMap, under ODbL">Carto</a>'
      },

      /*
       ** Props for l-reference-chart
       */
      l_reference_chart_props: {
        // Good acessibility map colours http://colorbrewer2.org/#type=diverging&scheme=RdYlGn&n=5
        colorScale: ["d7191c", "fdae61", "ffffbf", "a6d96a", "1a9641"],
        // min: "10",
        // max: "80",
        position: "topright"
      }
    };
  },
  methods: {
    zoomUpdated(zoom) {
      this.l_map_props.zoom = zoom;
      console.info("Zoom to: ", zoom);
    },
    centerUpdated(center) {
      this.l_map_props.center = center;
    },
    boundsUpdated(bounds) {
      this.l_map_props.bounds = bounds;
    },
    enteredAnArea() {
      console.log("pikachu", event);
    }
  }
};
</script>

<style lang="scss">
html,
body,
#app {
  height: 100%;
  margin: 0;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  color: #2c3e50;
  // margin-top: 60px;
}

h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

path.leaflet-interactive:not(:hover) {
  fill: transparent;
}

path.leaflet-interactive {
  stroke: currentColor;
}

.leaflet-popup-content-wrapper {
  border-radius: 0px;
  background-color: black;
  color: white;
  font-size: 20px;
}

.leaflet-popup-content {
  margin: 5px;
}

.leaflet-popup-tip {
  background: black;
}

.leaflet-popup-close-button {
  display: none;
}

.c-icon--london {
  border-bottom: 5px tomato solid;
}

.c-icon--paris {
  border-bottom: 10px dotted deepskyblue;
}
</style>
