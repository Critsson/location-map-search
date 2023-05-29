<template>
  <div>
    <el-button type="primary" size="large" :icon="Position" @click="getCoords"
      >Current Location</el-button
    >
  </div>
</template>

<script setup>
import { Position } from "@element-plus/icons-vue";
import { ref } from "vue";

const coords = ref({ lat: 0, lng: 0 });

const emit = defineEmits(["getLocation"]);

const getCoords = () => {
  navigator.geolocation.getCurrentPosition((pos) => {
    const crd = pos.coords;
    coords.value.lat = crd.latitude;
    coords.value.lng = crd.longitude;
    emit("getLocation", coords.value);
  });
};
</script>

<style scoped></style>
