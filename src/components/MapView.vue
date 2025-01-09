<template>
  <div class="map-container">
    <button class="gps-button" @click="focusOnLocation">
      Open My Location
    </button>
    <div id="map" class="map"></div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import 'ol/ol.css';
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import Geolocation from 'ol/Geolocation';
import { fromLonLat } from 'ol/proj';
import { Icon, Style } from 'ol/style';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';

export default {
  name: 'MapView',
  setup() {
    const map = ref(null);
    const geolocation = ref(null);
    const userPosition = ref(null);
    const userMarker = ref(null);

    const addMarker = (position) => {
      const feature = new Feature({
        geometry: new Point(position),
      });
      feature.setStyle(
        new Style({
          image: new Icon({
            src: 'https://cdn-icons-png.flaticon.com/512/64/64113.png',
            scale: 0.05,
          }),
        })
      );

      if (!userMarker.value) {
        userMarker.value = new VectorLayer({
          source: new VectorSource({
            features: [feature],
          }),
        });
        map.value.addLayer(userMarker.value);
      } else {
        userMarker.value.getSource().clear();
        userMarker.value.getSource().addFeature(feature);
      }
    };

    const focusOnLocation = () => {
      if (userPosition.value) {
        const position = fromLonLat(userPosition.value);
        map.value.getView().setCenter(position);
        map.value.getView().setZoom(5);
        addMarker(position);
      } else {
        alert("GPS Location not available.");
      }
    };

    onMounted(() => {
      map.value = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            source: new OSM(),
          }),
        ],
        view: new View({
          center: [0, 0],
          zoom: 3,
          maxZoom: 18, // Batasi zoom maksimum
          minZoom: 2, // Batasi zoom minimum
        }),
      });

      geolocation.value = new Geolocation({
        tracking: true,
        projection: map.value.getView().getProjection(),
        enableHighAccuracy: true,
      });

      geolocation.value.on('change', () => {
        const position = geolocation.value.getPosition();
        console.log('Raw Position', position);
        userPosition.value = position;
      });

      geolocation.value.on('change:accuracy', () => {
        const accuracy = geolocation.value.getAccuracy();
        console.log(`Location accuracy: ${accuracy} meters`);
        if (accuracy > 50) {
          alert('Warning: GPS location may not be accurate.');
        }
      });

      geolocation.value.on('error', (error) => {
        console.error('Geolocation error:', error.message);
      });
    });

    return { focusOnLocation };
  },
};
</script>

<style scoped>
.map-container {
  position: relative;
  width: 100%;
  height: 100vh;
}

.map {
  width: 100%;
  height: 100%;
}

.gps-button:hover {
  background-color: #0056b3;
}
</style>