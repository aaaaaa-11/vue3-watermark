<template>
  <div class="upload-wrap">
    <p>上传图片：</p>
    <div class="box" @click="uploadImg">
      <input ref="input" type="file" accept="image/*" @change="changeFile">
      <span v-show="!imgUrl" class="upload-icon">+</span>
      <p v-show="!imgUrl">上传</p>
      <img v-show="imgUrl" class="img" :src="imgUrl" alt="" srcset="" crossorigin="anonymous">
    </div>
  </div>
  <hr>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const props = defineProps<{
  imgUrl: string
}>()

const emit = defineEmits(['update:imgUrl'])

const input = ref<HTMLInputElement>()
const imgUrl = computed(() => props.imgUrl)

// 点击上传图片
const uploadImg = () => {
  input.value?.click()
}

// 转base64
function getBase64(file: File) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result);
    reader.onerror = error => reject(error);
  });
}

// 文件修改了
const changeFile = async (e: any) => {
  const url = (await getBase64(e.target?.files[0]) as string);
  emit('update:imgUrl', url)
}
</script>

<style lang="less" scoped>
.upload-wrap {
  padding: 1rem 1.5rem;
  display: flex;
  .box {
    border: 1px solid rgb(142, 142, 142);
    width: 120px;
    height: 120px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    position: relative;
    background-color: white;
    cursor: pointer;
    input {
      width: 0;
      height: 0;
    }
    .upload-icon {
      font-size: 32px;
      height: 32px;
      line-height: 32px;
    }
    .img {
      width: 119px;
      height: 119px;
    }
  }
}
</style>