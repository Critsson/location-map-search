<template>
  <div>
    <el-table
      :data="props.searchedPlaces"
      @selection-change="handleMultipleSelection"
    >
      <el-table-column type="selection" />
      <el-table-column prop="name" label="Name" width="350" />
      <el-table-column prop="lat" label="Latitude" width="200" />
      <el-table-column prop="lng" label="Longitude" width="200" />
    </el-table>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps(["searchedPlaces"]);
const multipleSelection = ref([]);

const handleMultipleSelection = (newSelection) => {
  multipleSelection.value = newSelection;
  console.log(multipleSelection.value);
};
</script>

<style scoped></style>
