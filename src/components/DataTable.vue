<template>
  <div class="table-container">
    <div style="width: 100%; display: flex; justify-content: flex-end">
      <el-button
        :icon="Delete"
        @click="handleDelete"
        type="danger"
        v-if="multipleSelection.length > 0"
        >Delete</el-button
      >
      <el-button :icon="Delete" v-else disabled>Delete</el-button>
    </div>
    <el-table
      :data="props.paginated[page - 1]"
      @selection-change="handleMultipleSelection"
      border
      highlight-current-row
      width="100%"
    >
      <el-table-column type="selection" />
      <el-table-column prop="name" label="Name" width="250" />
      <el-table-column prop="lat" label="Latitude" width="160" />
      <el-table-column prop="lng" label="Longitude" width="160" />
    </el-table>
    <div style="width: 100%; display: flex; justify-content: center">
      <el-pagination
        layout="prev, pager, next"
        :total="props.searchedPlaces.length"
        background
        v-model:current-page="page"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { Delete } from "@element-plus/icons-vue";

const props = defineProps(["searchedPlaces", "paginated"]);
const emit = defineEmits(["delete"]);
const multipleSelection = ref([]);
const page = ref(1);

const handleMultipleSelection = (newSelection) => {
  multipleSelection.value = newSelection;
};

const handleDelete = () => {
  emit("delete", multipleSelection.value);
};
</script>

<style scoped>
.table-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-left: 1vw;
  margin-top: -30px;
}
</style>
