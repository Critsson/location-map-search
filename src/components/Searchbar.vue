<template>
  <div class="search-container">
    <el-input
      @keydown.enter="onSearch"
      size="large"
      v-model="searchTerm"
      placeholder="Enter Location"
    />
    <el-button size="large" :icon="Search" @click="onSearch">Search</el-button>
  </div>
</template>

<script setup>
import { Search } from "@element-plus/icons-vue";
import { ref } from "vue";

const emit = defineEmits(["search"]);

const searchTerm = ref("");

const onSearch = async () => {
  emit("search", searchTerm.value);
  searchTerm.value = "";
};
</script>

<style scoped>
.search-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}
</style>
