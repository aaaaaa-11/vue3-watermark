<template>
  <a-row class="watermark-wrap">
    <a-col class="watermark-config">
      <UploadImg v-model:imgUrl="imgUrl" />
      <label v-show="imgUrl" title="修改后会清空所有水印" @change="draw">
        图片是否缩放：
        <a-radio-group v-model:value="isScaling">
          <a-radio-button :value="false">图片原来的尺寸</a-radio-button>
          <a-radio-button :value="true">适应浏览器宽度</a-radio-button>
        </a-radio-group>
      </label>
      <a-form
        v-show="imgUrl"
        class="watermark-form"
        :model="markOption"
        ref="formRef"
        :label-col="{ span: 10 }"
        :wrapper-col="{ span: 12 }"
        autocomplete="off"
        @finish="addWaterMark"
      >
        <a-form-item label="水印文字" name="text" :rules="[{ required: true }]">
          <a-input type="text" v-model:value="markOption.text" placeholder="请输入水印文字" />
        </a-form-item>
        <a-form-item label="水印颜色" name="color" :rules="[{ required: true }]">
          <v3-color-picker
            size="small"
            v-model:value="markOption.color"
            theme="light"
            :width="300"
          ></v3-color-picker>
        </a-form-item>
        <a-form-item label="水印背景颜色" name="bgColor" :rules="[{ required: true }]">
          <v3-color-picker
            size="small"
            v-model:value="markOption.bgColor"
            theme="light"
            :width="300"
          ></v3-color-picker>
        </a-form-item>
        <a-form-item label="文字大小：" name="fontSize" :rules="[{ required: true }]">
          <a-input type="number" v-model:value="markOption.fontSize" />
        </a-form-item>
        <a-form-item
          label="文字字重（font-weight）"
          name="fontWeight"
          :rules="[{ required: true }]"
        >
          <a-input type="text" v-model:value="markOption.fontWeight" />
        </a-form-item>
        <a-form-item label="字体格式（font-family)" name="fontFamily" :rules="[{ required: true }]">
          <a-input type="text" v-model:value="markOption.fontFamily" />
        </a-form-item>
        <a-form-item label="文本基准线" name="textBaseline" :rules="[{ required: true }]">
          <a-input type="text" v-model:value="markOption.textBaseline" />
        </a-form-item>
        <a-form-item label="文字横向位置：" name="left" :rules="[{ required: true }]">
          <a-input type="number" v-model:value="markOption.left" />
        </a-form-item>
        <a-form-item label="文字纵向位置：" name="top" :rules="[{ required: true }]">
          <a-input type="number" v-model:value="markOption.top" />
        </a-form-item>
        <a-form-item label="旋转角度：" name="rotate" :rules="[{ required: true }]">
          <a-input type="number" v-model:value="markOption.rotate" :min="-180" :max="180" />
        </a-form-item>
        <div class="btn-wrap">
          <a-space>
            <a-button @click="clearText">清空所有文字</a-button>
            <a-button @click="undo(1)">清除上一次添加的文字</a-button>
            <a-button type="primary" html-type="submit">确认</a-button>
          </a-space>
        </div>
      </a-form>
    </a-col>
    <a-col class="watermark-img" v-show="imgUrl">
      <p class="tip">若要保存图片，可在图片上右键 -> 图片存储为 -> ...</p>
      <a-divider />
      <canvas class="canvas" ref="canvas" :width="CANVAS_WIDTH" :height="CANVAS_HEIGHT"></canvas>
      <img :src="imgUrl" alt="" ref="img" @load="draw" class="canvas-img" crossorigin="anonymous" />
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import type { FormInstance } from 'ant-design-vue'
import { message } from 'ant-design-vue'
import { V3ColorPicker } from 'v3-color-picker'
import type { MarkOption } from '../../types/index.d.ts'
import UploadImg from './UploadImg.vue'

const DEFAULT_WIDTH = 700
const innerWidth = window.innerWidth
const fixScreenWidth = innerWidth - 500
const CANVAS_WIDTH = fixScreenWidth > DEFAULT_WIDTH ? fixScreenWidth : DEFAULT_WIDTH
const CANVAS_HEIGHT = Math.round(CANVAS_WIDTH * 0.8)

const canvas = ref<HTMLCanvasElement>()
const img = ref<HTMLImageElement>()
const imgUrl = ref<string>('')
const isScaling = ref<boolean>(true)

const markOption = reactive<MarkOption>({
  text: null,
  color: '#000000',
  bgColor: '#00000000',
  fontSize: 100,
  textBaseline: 'top',
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
  const url = imgUrl.value
  if (!url) return
  const imgEl = img.value
  const canvasEl = canvas.value
  if (canvasEl && imgEl) {
    let height: number = CANVAS_HEIGHT
    let width: number = CANVAS_WIDTH

    const ctx = (canvasEl as HTMLCanvasElement).getContext('2d')
    // 清空画布和缓存
    ctx?.clearRect(0, 0, canvasEl.width, canvasEl.height)
    imgStore = []

    const imgH = imgEl.naturalHeight
    const imgW = imgEl.naturalWidth

    const scaling = isScaling.value
    if (scaling) {
      // 根据屏幕大小缩放
      if (imgW / imgH < 1) {
        height = CANVAS_HEIGHT
        width = (imgW * CANVAS_HEIGHT) / imgH
      } else {
        width = CANVAS_WIDTH
        height = (imgH * CANVAS_WIDTH) / imgW
      }
    } else {
      // 图片原尺寸
      width = imgW
      height = imgH
    }

    // scaleX = width / imgW
    // scaleY = height / imgH
    if (width && height) {
      canvasEl.height = height
      canvasEl.width = width
    }
    imgEl.width = width
    imgEl.height = height
    ctx?.drawImage(imgEl, 0, 0, width, height)
    saveImgData()
  }
}

// 上一步
const undo = (step: number = 1) => {
  const canvasEl = canvas.value
  const ctx = (canvasEl as HTMLCanvasElement).getContext('2d')
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
    const ctx = canvasEl.getContext('2d')
    if (ctx) {
      ctx.font = `${markOption.fontWeight} ${markOption.fontSize}px ${markOption.fontFamily}`
      ctx.rotate((markOption.rotate * Math.PI) / 180)
      ctx.fillStyle = markOption.bgColor
      const { width } = ctx.measureText(markOption.text as string)
      ctx.fillRect(markOption.left, markOption.top, width, +markOption.fontSize)
      ctx.textBaseline = markOption.textBaseline as CanvasTextBaseline
      ctx.fillStyle = markOption.color
      ctx.fillText(markOption.text as string, markOption.left, markOption.top)
      saveImgData()
      ctx.rotate((-markOption.rotate * Math.PI) / 180) // 反向还原角度
    }
  }
}
</script>

<style lang="less" scoped>
.watermark-wrap {
  padding: 2rem;
  min-width: 1240px;
  .watermark-config {
    width: 400px;
    .watermark-form {
      border-top: 1px solid #efefef;
      padding-top: 2rem;
      margin-top: 2rem;

      .btn-wrap {
        text-align: center;
      }
    }
  }
  .watermark-img {
    flex: 1;
    margin-left: 1rem;
    position: relative;
    .tip {
      color: gray;
    }
    .canvas-img {
      position: absolute;
      z-index: -1;
      opacity: 0;
      left: 0;
      bottom: 0;
    }
  }
}
</style>
