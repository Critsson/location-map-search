<template>
  <div>
    <el-input
      @keydown.enter="onSearch"
      size="large"
      v-model="searchTerm"
      placeholder="Search Location"
    />
  </div>
</template>

<script setup>
import { ref, watchEffect } from "vue";

const emit = defineEmits(["search"]);

const searchTerm = ref("");

const onSearch = async () => {
  emit("search", searchTerm.value);
};

watchEffect(() => {
  console.log(searchTerm.value);
});
</script>

<style scoped></style>
