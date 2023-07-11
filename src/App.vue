<template>
  <UploadImg v-model:imgUrl="imgUrl" />
  <WaterMark :imgUrl="imgUrl" v-show="imgUrl" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import UploadImg from './components/UploadImg.vue';
import WaterMark from './components/WaterMark.vue';

const imgUrl = ref<string>('')

</script>