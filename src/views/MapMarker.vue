<template>
  <!--Création de la map avec Leaflet-->
  <l-map
      :center="center"
      :zoom="zoom"
      ref="map"
      class="z-10"
  >
    <l-tile-layer :url="url"></l-tile-layer>
    <!--Si enabledBike est vrai alors on affiche les stations de vélos sur la map-->
    <template v-if="enabledBike">
      <!-- Bouclage sur les données récupérées avec l'api pour pouvoir placés les marqueurs-->
      <l-marker
          v-for="marker in markersBike"
          :key="marker.id"
          :lat-lng="[marker.lat , marker.lon ]"
      >
        <!--Mise en place de l'icon unique pour les stations de vélos-->
        <l-icon :icon-size="[35, 27]"
                icon-url="/img/parkVelo.png">
        </l-icon>
        <!--Pop Up avec les donnéés des stations vélos-->
        <l-popup> Nombre de vélo disponible : {{ marker.bike_available }} <br>
          Places disponible : {{ marker.nbr_available }} <br>
          Pourcentage de vélo restant : {{ marker.pct_available }} %
        </l-popup>
      </l-marker>
    </template>

<!--Si enabledCar est vrai alors on affiche les marqueurs des voitures-->
    <template v-if="enabledCar">
      <l-marker
          v-for="marker in markersCar"
          :key="marker.id"
          :lat-lng="[marker.lat , marker.lon ]"
      >
<!--Mise en place des icons unique pour les voitures-->
        <l-icon :icon-size="[45, 35]"
                icon-url="/img/parkVoiture.png">
        </l-icon>
<!--Mise en place des PopUps avec les données requises-->
        <l-popup>
          Nom Parking : {{ marker.name }} <br>
          Nombre de place de parkings disponible : {{ marker.nbr_space_available }} <br>
          Capacité : {{ marker.nbr_total_space }} <br>
          Vehicule autorisé : {{ marker.authorized_car_type }}
        </l-popup>
      </l-marker>
    </template>
<!--Si EnabledBus est vrai alors -->
    <template v-if="enabledBus">
      <l-marker-cluster-group>
        <l-marker
            v-for="marker in markersBus"
            :key="marker.id"
            :lat-lng="[marker.lat , marker.lon]"
        >
          <l-popup>
            Nom Arret de Bus : {{ marker.name }} <br>
          </l-popup>
          <l-icon :icon-size="[45, 35]"
                  icon-url="/img/logoBus.png">
          </l-icon>
        </l-marker>
      </l-marker-cluster-group>
    </template>

    <template v-if="enabledTram">
      <l-marker
          v-for="marker in markersTram"
          :key="marker.id"
          :lat-lng="[marker.lat , marker.lon ]">
        <l-icon :icon-size="[50, 40]"
                icon-url="/img/logoTram.png">
        </l-icon>
        <l-popup>
          Nom Arret : {{ marker.name }} <br>
          Refresh : {{ marker.updated_at }} <br>
        </l-popup>
      </l-marker>

      <l-polyline
          v-for="(marker,i) in markersTramCoordinates.results"
          :key="i"
          :lat-lngs="marker.geo_shape.geometry.coordinates" :color='marker.code_coule'
      >
      </l-polyline>
    </template>
  </l-map>
  <div
      class="z-50 ml-4 w-60 rounded-md bg-neutral-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm hover:bg-neutral-400 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-neutral-400 px-3.5 py-2.5 sticky z-30 -mt-60 ">
    <div class="flex mb-2">
      <Switch v-model="enabledCar"
              :class="[enabledCar ? 'bg-indigo-600' : 'bg-gray-200', 'relative inline-flex h-6 w-11 flex-shrink-0 cursor-pointer rounded-full border-2 border-transparent transition-colors duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-indigo-600 focus:ring-offset-2']">
        <span class="sr-only">Afficher les Places de Parkings</span>
        <span aria-hidden="true"
              :class="[enabledCar ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none inline-block h-5 w-5 transform rounded-full bg-white shadow ring-0 transition duration-200 ease-in-out']"/>
      </Switch>
      <p> Parking Voiture </p>
    </div>

    <div class="flex mb-2">
      <Switch v-model="enabledBike"
              @click="refresh"
              :class="[enabledBike ? 'bg-indigo-600' : 'bg-gray-200', 'relative inline-flex h-6 w-11 flex-shrink-0 cursor-pointer rounded-full border-2 border-transparent transition-colors duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-indigo-600 focus:ring-offset-2']">
        <span class="sr-only">Afficher les stations Vélomagg</span>
        <span aria-hidden="true"
              :class="[enabledBike ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none inline-block h-5 w-5 transform rounded-full bg-white shadow ring-0 transition duration-200 ease-in-out']"/>
      </Switch>
      <p> Parking velo </p>
    </div>

    <div class="flex mb-2">
      <Switch v-model="enabledBus"
              :class="[enabledBus ? 'bg-indigo-600' : 'bg-gray-200', 'relative inline-flex h-6 w-11 flex-shrink-0 cursor-pointer rounded-full border-2 border-transparent transition-colors duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-indigo-600 focus:ring-offset-2']">
        <span class="sr-only">Afficher les stations Bus</span>
        <span aria-hidden="true"
              :class="[enabledBus ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none inline-block h-5 w-5 transform rounded-full bg-white shadow ring-0 transition duration-200 ease-in-out']"/>
      </Switch>
      <p> Parking Bus </p>
    </div>

    <div class="flex">
      <Switch v-model="enabledTram"
              :class="[enabledTram ? 'bg-indigo-600' : 'bg-gray-200', 'relative inline-flex h-6 w-11 flex-shrink-0 cursor-pointer rounded-full border-2 border-transparent transition-colors duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-indigo-600 focus:ring-offset-2']">
        <span class="sr-only">Afficher les stations de tram</span>
        <span aria-hidden="true"
              :class="[enabledTram ? 'translate-x-5' : 'translate-x-0', 'pointer-events-none inline-block h-5 w-5 transform rounded-full bg-white shadow ring-0 transition duration-200 ease-in-out']"/>
      </Switch>
      <p> Parking Tram </p>
    </div>
  </div>
</template>

<script setup>
import {ref} from 'vue'
import {LPolyline, LIcon, LMap, LMarker, LPopup, LTileLayer} from '@vue-leaflet/vue-leaflet';
import {LMarkerClusterGroup} from "vue-leaflet-markercluster";
import 'leaflet/dist/leaflet.css';
import 'vue-leaflet-markercluster/dist/style.css'
import axios from "axios";
import {Switch} from '@headlessui/vue'


const url = 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
const center = ref([43.5974, 3.8820])
const zoom = ref(12)
const markersBike = ref([])
const markersCar = ref([])
const markersBus = ref([])
const markersTram = ref([])
const markersTramCoordinates = ref([])
const enabledCar = ref(false)
const enabledBike = ref(true)
const enabledTram = ref(false)
const enabledBus = ref(true)

axios.get('http://localhost/api/bike/station').then((response) => {
  markersBike.value = response.data
});
axios.get('http://localhost/api/car/parking').then((response) => {
  markersCar.value = response.data
})
axios.get('http://localhost/api/bus/stop').then((response) => {
  markersBus.value = response.data
});

axios.get('http://localhost/api/tram/coordinates').then((response) => {
  console.log(response.data)

  markersTramCoordinates.value = response.data
});
axios.get('http://localhost/api/tram/stop').then((response) => {
  markersTram.value = response.data
})

function refresh() {
  axios.get('http://localhost/api/bike/station').then((response) => {
    markersBike.value = response.data
  })
}
</script>

<style>

.map {
  width: 80%;
  height: 100%;
}
</style>
