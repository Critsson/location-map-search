<script setup>
import LocationButton from "./components/LocationButton.vue";
import Searchbar from "./components/Searchbar.vue";
import DataTable from "./components/DataTable.vue";
import { onMounted, ref } from "vue";

const mapDiv = ref(null);
const searchedPlaces = ref(
  localStorage.getItem("searchedPlaces")
    ? JSON.parse(localStorage.getItem("searchedPlaces"))
    : []
);
let map;
const searchedPlacesMarkers = ref({});

const addCurrentLocation = async (coords) => {
  const { Marker, Animation } = await google.maps.importLibrary("marker");
  searchedPlaces.value.push({ ...coords, name: "Current Location" });
  localStorage.setItem("searchedPlaces", JSON.stringify(searchedPlaces.value));
  searchedPlacesMarkers.value["Current Location"] = new Marker({
    position: coords,
    map,
    animation: Animation.DROP,
    title: "Current Location",
  });
  map.setCenter(results[0].geometry.location);
  map.setZoom(10);
};

onMounted(async () => {
  const { Marker, Animation } = await google.maps.importLibrary("marker");
  const { Map } = await google.maps.importLibrary("maps");
  map = new Map(mapDiv.value, {
    center: { lat: 0, lng: 0 },
    zoom: 2,
  });
  for (let i = 0; i < searchedPlaces.value.length; i++) {
    searchedPlacesMarkers.value[searchedPlaces.value[i].name] = new Marker({
      position: {
        lat: searchedPlaces.value[i].lat,
        lng: searchedPlaces.value[i].lng,
      },
      map,
      animation: Animation.DROP,
      title: searchedPlaces.value[i].name,
    });
  }
});

const handleSearch = async (searchTerm) => {
  const { PlacesService } = await google.maps.importLibrary("places");
  const { Marker, Animation } = await google.maps.importLibrary("marker");
  const request = {
    query: searchTerm,
    fields: ["name", "geometry"],
  };
  const service = new PlacesService(map);

  service.findPlaceFromQuery(request, (results, status) => {
    searchedPlaces.value.push({
      name: results[0].name,
      lat: results[0].geometry.location.lat(),
      lng: results[0].geometry.location.lng(),
    });
    localStorage.setItem(
      "searchedPlaces",
      JSON.stringify(searchedPlaces.value)
    );
    console.log(searchedPlaces.value);

    searchedPlacesMarkers.value[results[0].name] = new Marker({
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
  });
};
</script>

<template>
  <div class="main-container">
    <LocationButton @get-location="(coords) => addCurrentLocation(coords)" />
    <Searchbar @search="(searchTerm) => handleSearch(searchTerm)" />
    <div
      ref="mapDiv"
      style="width: 40vw; height: 60vh; border-radius: 1vw"
    ></div>
    <DataTable :searchedPlaces="searchedPlaces" />
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
}
</style>
