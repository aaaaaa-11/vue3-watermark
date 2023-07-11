<template>
  <a-row class="row">
    <a-col>
      <canvas class="canvas" ref="canvas" :width="CANVAS_WIDTH" :height="CANVAS_HEIGHT"></canvas>
      <img :src="imgUrl" alt="" ref="img" @load="draw" class="canvas-img" crossorigin="anonymous">
      <a-divider />
      <p>若要保存图片，可在图片上右键 -> 图片存储为 -> ...</p>
    </a-col>
    <a-col class="col right">
      <a-form 
        :model="markOption"
        name="basic"
        ref="formRef"
        :label-col="{ span: 8 }"
        :wrapper-col="{ span: 16 }"
        autocomplete="off"
        @finish="addWaterMark">
        <a-form-item label="水印文字" name="text" :rules="[{required: true}]">
          <a-input type="text" v-model:value="markOption.text" placeholder="请输入水印文字" />
        </a-form-item>
        <a-form-item label="水印颜色" name="color" :rules="[{required: true}]">
          <v3-color-picker size="small" v-model:value="markOption.color" theme="light" :width="300"></v3-color-picker>
        </a-form-item>
        <a-form-item label="文字大小：" name="fontSize" :rules="[{required: true}]">
          <a-input type="number" v-model:value="markOption.fontSize" />
        </a-form-item>
        <a-form-item label="文字字重（font-weight）" name="fontWeight" :rules="[{required: true}]">
          <a-input type="text" v-model:value="markOption.fontWeight" />
        </a-form-item>
        <a-form-item label="字体格式（font-family)" name="fontFamily" :rules="[{required: true}]">
          <a-input type="text" v-model:value="markOption.fontFamily" />
        </a-form-item>
        <a-form-item label="文字横向位置：" name="left" :rules="[{required: true}]">
          <a-input type="number" v-model:value="markOption.left" />
        </a-form-item>
        <a-form-item label="文字纵向位置：" name="top" :rules="[{required: true}]">
          <a-input type="number" v-model:value="markOption.top" />
        </a-form-item>
        <a-form-item label="旋转角度：" name="rotate" :rules="[{required: true}]">
          <a-input type="number" v-model:value="markOption.rotate" :min="-180" :max="180" />
        </a-form-item>
        <a-form-item :wrapper-col="{ offset: 8 }">
          <a-space>
            <a-button @click="clearText">清空所有文字</a-button>
            <a-button @click="undo(1)">清除上一次添加的文字</a-button>
            <a-button type="primary" html-type="submit">确认</a-button>
          </a-space>
        </a-form-item>
      </a-form>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { ref, reactive, watch, onMounted } from 'vue'
import type { FormInstance } from 'ant-design-vue';
import { message } from 'ant-design-vue'
import { V3ColorPicker } from 'v3-color-picker';
import type { MarkOption } from '../../types/index.d.ts'

const CANVAS_WIDTH = 400
const CANVAS_HEIGHT = 300

const props = defineProps<{
  imgUrl: string
}>()

const canvas = ref<HTMLCanvasElement>()
const img = ref<HTMLImageElement>()

const markOption = reactive<MarkOption>({
  text: null,
  color: '#000000',
  fontSize: 100,
  fontFamily: '微软雅黑',
  fontWeight: 'normal',
  rotate: 45,
  left: 0,
  top: 0
})

const formRef = ref<FormInstance>()

let imgStore: any = []
const saveImgData = () => {
  const canvasEl = canvas.value
  if (canvasEl) {
    const ctx = canvasEl.getContext('2d')
    ctx && imgStore.push(ctx.getImageData(0, 0, canvasEl.width, canvasEl.height))
  }
}

// 上传图片后在canvas中绘制图片
const draw = () => {
  console.log('draw');

  const url = props.imgUrl
  if (!url) return
  const imgEl = img.value
  const canvasEl = canvas.value
  if (canvasEl && imgEl) {
    let height: number = CANVAS_HEIGHT
    let width: number = CANVAS_WIDTH

    const ctx = (canvasEl as HTMLCanvasElement).getContext("2d")
    // 清空画布和缓存
    ctx?.clearRect(0, 0, canvasEl.width, canvasEl.height)
    imgStore = []

    const imgH = imgEl.naturalHeight
    const imgW = imgEl.naturalWidth

    if (imgW / imgH < 1) {
      height = canvasEl.height
      width = imgW * canvasEl.height / imgH
    } else {
      width = canvasEl.width
      height = imgH * canvasEl.width / imgW
    }
    // scaleX = width / imgW
    // scaleY = height / imgH
    if (width && height) {
      canvasEl.height = height
      canvasEl.width = width
    }
    ctx?.drawImage(imgEl, 0, 0);
    saveImgData()
  }
}

// 上一步
const undo = (step: number = 1) => {
  const canvasEl = canvas.value
  const ctx = (canvasEl as HTMLCanvasElement).getContext("2d")
  let len = imgStore.length
  if (len > 1) {
    ctx?.putImageData(imgStore[len - 1 - step], 0, 0)
    imgStore.length = len - step
  } else {
    message.warn('没有可清除的文字了')
  }
}

const clearText = () => {
  undo(imgStore.length - 1)
}

// 添加水印
const addWaterMark = () => {
  const canvasEl = canvas.value
  if (canvasEl) {
    const ctx = canvasEl.getContext("2d");
    if (ctx) {
      ctx.fillStyle = markOption.color;
      ctx.textBaseline = "middle";
      ctx.font = `${markOption.fontWeight} ${markOption.fontSize}px ${markOption.fontFamily}`;
      ctx.rotate(markOption.rotate * Math.PI/180)
      ctx.fillText((markOption.text as string), markOption.left, markOption.top);
      saveImgData()
      ctx.rotate(-markOption.rotate * Math.PI/180) // 反向还原角度
    }
  }
}

onMounted(() => {
  // draw()
})

watch(
  () => props.imgUrl,
  (url: string) => {
    // draw()
  }
)
</script>

<style lang="less" scoped>
.row {
  padding: 1rem 2rem;
  .canvas {
    border: 1px solid rgb(142, 142, 142);
  }
  .canvas-img {
    position: absolute;
    z-index: -1;
    opacity: 0;
  }
  .right {
    flex: 1;
  }
}
</style>