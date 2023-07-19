<template>
  <div class="upload-wrap">
    <div class="label">
      <p>上传图片：</p>
      <p>（不会上传服务器）</p>
    </div>
    <div class="box" @click="uploadImg">
      <input ref="input" type="file" accept="image/*" @change="changeFile">
      <span v-show="!imgUrl" class="upload-icon">+</span>
      <p v-show="!imgUrl">上传</p>
      <img v-show="imgUrl" class="img" :src="imgUrl" alt="" srcset="" crossorigin="anonymous">
    </div>
  </div>
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
  display: flex;
  margin-bottom: 2rem;
  .label {
    text-align: right;
    width: 189px;
  }
  .box {
    border: 1px dashed rgb(215, 215, 215);
    width: 120px;
    height: 120px;
    border-radius: 2px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    position: relative;
    background-color: white;
    color: gray;
    cursor: pointer;
    input {
      width: 0;
      height: 0;
    }
    .upload-icon {
      font-size: 32px;
      line-height: 36px;
      color: rgb(160, 160, 160);
    }
    .img {
      width: 119px;
      height: 119px;
    }
  }
}
</style>