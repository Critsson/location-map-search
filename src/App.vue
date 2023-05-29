<script setup>
import LocationButton from "./components/LocationButton.vue";
import Searchbar from "./components/Searchbar.vue";
import DataTable from "./components/DataTable.vue";
import { computed, onMounted, ref } from "vue";
import axios from "axios";
//localStorage.clear();
const mapDiv = ref(null);
const searchedPlaces = ref(
  localStorage.getItem("searchedPlaces")
    ? JSON.parse(localStorage.getItem("searchedPlaces"))
    : []
);
const id = ref(
  localStorage.getItem("searchedPlaces")
    ? JSON.parse(localStorage.getItem("searchedPlaces"))[
        JSON.parse(localStorage.getItem("searchedPlaces")).length - 1
      ].id + 1
    : 1
);
const timeZone = ref("");
const localTime = ref(0);
const searchedPlacesMarkers = ref({});
const timestamp = ref(Date.now() / 1000);
const latLngQuery = computed(() => {
  if (searchedPlaces.value.length > 0) {
    const { lat, lng } = searchedPlaces.value[searchedPlaces.value.length - 1];
    return `${lat}%2C${lng}`;
  }
});
let map;
const paginatedSearchedPlaces = computed(() => {
  const placeholder = [];
  let arr = [];
  for (let i = 0; i < searchedPlaces.value.length; i++) {
    arr.push(searchedPlaces.value[i]);
    if ((i + 1) % 10 === 0) {
      placeholder.push([...arr]);
      arr = [];
    }
  }
  if (arr.length < 10) {
    placeholder.push([...arr]);
    arr = [];
  }
  return placeholder;
});

const getLocalTime = async () => {
  try {
    const res = await axios.get(
      `https://maps.googleapis.com/maps/api/timezone/json?location=${latLngQuery.value}&timestamp=${timestamp.value}&key=AIzaSyACkPS4wrZCYTAdFQUMUotR1Yfx3P86WBs`
    );
    const localMilli = new Date(timestamp.value * 1000).toLocaleString(
      "en-US",
      {
        timeZone: res.data.timeZoneId,
      }
    );
    localTime.value = localMilli.split(", ")[1];
    timeZone.value = res.data.timeZoneName;
  } catch (error) {}
};

const addCurrentLocation = async (coords) => {
  const { Marker, Animation } = await google.maps.importLibrary("marker");
  searchedPlaces.value.push({
    ...coords,
    name: "Current Location",
    id: id.value,
  });
  localStorage.setItem("searchedPlaces", JSON.stringify(searchedPlaces.value));
  getLocalTime();
  searchedPlacesMarkers.value[id.value] = new Marker({
    position: coords,
    map,
    animation: Animation.DROP,
    title: "Current Location",
  });
  map.setCenter(coords);
  map.setZoom(10);
  id.value++;
};

onMounted(async () => {
  const { Marker, Animation } = await google.maps.importLibrary("marker");
  const { Map } = await google.maps.importLibrary("maps");
  getLocalTime();
  map = new Map(mapDiv.value, {
    center: { lat: 0, lng: 0 },
    zoom: 2,
  });
  if (searchedPlaces.value.length > 0) {
    for (let i = 0; i < searchedPlaces.value.length; i++) {
      searchedPlacesMarkers.value[searchedPlaces.value[i].id] = new Marker({
        position: {
          lat: searchedPlaces.value[i].lat,
          lng: searchedPlaces.value[i].lng,
        },
        map,
        animation: Animation.DROP,
        title: searchedPlaces.value[i].name,
      });
    }
  }
});

const handleDelete = async (multipleSelection) => {
  for (let i = 0; i < multipleSelection.length; i++) {
    searchedPlaces.value = searchedPlaces.value.filter(
      (place) => place.id !== multipleSelection[i].id
    );
    localStorage.setItem(
      "searchedPlaces",
      JSON.stringify(searchedPlaces.value)
    );
    if (searchedPlaces.value.length < 1) {
      localStorage.removeItem("searchedPlaces");
    }
    let marker = searchedPlacesMarkers.value[multipleSelection[i].id];
    marker.setVisible(false);
    marker.setMap(null);
    marker = null;
    searchedPlacesMarkers.value[multipleSelection[i].id] = null;
  }
};

const handleSearch = async (searchTerm) => {
  const { PlacesService } = await google.maps.importLibrary("places");
  const { Marker, Animation } = await google.maps.importLibrary("marker");
  const request = {
    query: searchTerm,
    fields: ["name", "geometry"],
  };
  const service = new PlacesService(map);

  service.findPlaceFromQuery(request, (results, status) => {
    if (results) {
      searchedPlaces.value.push({
        name: results[0].name,
        lat: results[0].geometry.location.lat(),
        lng: results[0].geometry.location.lng(),
        id: id.value,
      });
      localStorage.setItem(
        "searchedPlaces",
        JSON.stringify(searchedPlaces.value)
      );
      getLocalTime();
      searchedPlacesMarkers.value[id.value] = new Marker({
        position: {
          lat: results[0].geometry.location.lat(),
          lng: results[0].geometry.location.lng(),
        },
        map,
        animation: Animation.DROP,
        title: results[0].name,
      });
      map.setCenter(results[0].geometry.location);
      map.setZoom(10);
      id.value++;
    }
  });
};
</script>

<template>
  <div class="main-container">
    <LocationButton @get-location="(coords) => addCurrentLocation(coords)" />
    <Searchbar @search="(searchTerm) => handleSearch(searchTerm)" />
    <div class="map-table-container">
      <div style="display: flex; flex-direction: column; align-items: center">
        <div
          ref="mapDiv"
          style="width: 45vw; height: 500px; border-radius: 1vw"
        ></div>
        <p v-if="localTime && searchedPlaces.length > 0">
          {{ searchedPlaces[searchedPlaces.length - 1].name }}: {{ localTime }}
          {{ timeZone }}
        </p>
      </div>
      <DataTable
        :searchedPlaces="searchedPlaces"
        :paginated="paginatedSearchedPlaces"
        @delete="(multipleSelection) => handleDelete(multipleSelection)"
      />
    </div>
  </div>
</template>

<style scoped>
.main-container {
  margin-top: 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
  width: 100vw;
  padding-left: 3px;
  padding-right: 3px;
}
.map-table-container {
  display: flex;
  justify-content: center;
  width: 100%;
}
</style>
