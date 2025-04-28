<template>
  <view class="preview-container">
    <view v-if="isLoading" class="loading-mask">
      <view class="loading-spinner"></view>
      <text class="loading-text">处理中，请稍候...</text>
    </view>

    <canvas
      canvas-id="previewCanvas"
      :style="{ width: displayWidth + 'px', height: displayHeight + 'px' }"
      :width="displayWidth * pixelRatio"
      :height="displayHeight * pixelRatio"
    />

    <view class="slider-container">
      <text>模糊强度 ({{ blurAmount }})</text>
      <slider
        :value="blurAmount"
        min="0"
        max="20"
        step="1"
        @change="updateBlurAmount"
        activeColor="#1E90FF"
      />
    </view>

    <view class="slider-container">
      <text>模糊区域大小 ({{ blurAreaSize }}%)</text>
      <slider
        :value="blurAreaSize"
        min="10"
        max="100"
        step="5"
        @change="updateBlurAreaSize"
        activeColor="#1E90FF"
      />
    </view>

    <view class="slider-container">
      <text>模糊区域位置 ({{ blurAreaPosition }}%)</text>
      <slider
        :value="blurAreaPosition"
        min="0"
        max="100"
        step="1"
        @change="updateBlurAreaPosition"
        activeColor="#1E90FF"
      />
    </view>

    <view class="button-group">
      <button class="btn save-btn" @click="saveCanvas">💾 保存照片</button>
      <button class="btn back-btn" @click="goBack">🔙 返回首页</button>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

// 响应式数据
const imgPath = ref('')
const isLoading = ref(false)
const canvasId = 'previewCanvas'
const displayWidth = ref(300)
const displayHeight = ref(400)
const pixelRatio = ref(1)
const blurAmount = ref(8)
const blurAreaSize = ref(50)
const blurAreaPosition = ref(50)
const canvasContext = ref(null)
let imageInfo = null

onLoad(async (options) => {
  imgPath.value = decodeURIComponent(options.img || '')
  const sysInfo = uni.getSystemInfoSync()
  pixelRatio.value = sysInfo.pixelRatio || 1
  canvasContext.value = uni.createCanvasContext(canvasId)
  await processImage()
})

async function processImage() {
  if (!imgPath.value || !canvasContext.value) return
  isLoading.value = true

  try {
    imageInfo = await new Promise((resolve, reject) => {
      uni.getImageInfo({
        src: imgPath.value,
        success: resolve,
        fail: reject
      })
    })

    const sysInfo = uni.getSystemInfoSync()
    const maxWidth = sysInfo.windowWidth * 0.9
    const maxHeight = sysInfo.windowHeight * 0.6

    let ratio = Math.min(
      maxWidth / imageInfo.width,
      maxHeight / imageInfo.height
    )
    if (ratio > 1) ratio = 1

    displayWidth.value = imageInfo.width * ratio
    displayHeight.value = imageInfo.height * ratio

    drawCanvas()

  } catch (e) {
    console.error('图片处理失败:', e)
    uni.showToast({
      title: '图片处理失败',
      icon: 'none'
    })
  } finally {
    isLoading.value = false
  }
}

function drawCanvas() {
  canvasContext.value = uni.createCanvasContext(canvasId) // <-- 新加这行

  const ctx = canvasContext.value
  const cw = displayWidth.value * pixelRatio.value
  const ch = displayHeight.value * pixelRatio.value

  ctx.clearRect(0, 0, cw, ch)

  // 绘制原图
  ctx.drawImage(
    imgPath.value,
    0, 0,
    imageInfo.width, imageInfo.height,
    0, 0,
    displayWidth.value,
    displayHeight.value
  )

  const blurRadius = (blurAreaSize.value / 100) * Math.min(cw, ch) / 2
  const centerX = cw * (blurAreaPosition.value / 100)
  const centerY = ch / 2

  ctx.save()
  ctx.beginPath()
  ctx.arc(centerX, centerY, blurRadius, 0, Math.PI * 2)
  ctx.clip()

  const scale = 1 - (blurAmount.value / 50)
  const sw = displayWidth.value * scale
  const sh = displayHeight.value * scale
  const sx = (displayWidth.value - sw) / 2
  const sy = (displayHeight.value - sh) / 2

  ctx.drawImage(
    imgPath.value,
    0, 0,
    imageInfo.width, imageInfo.height,
    sx, sy,
    sw,
    sh
  )

  ctx.restore()

  ctx.draw()
}


function updateBlurAmount(e) {
  blurAmount.value = e.detail.value
  drawCanvas()
}

function updateBlurAreaSize(e) {
  blurAreaSize.value = e.detail.value
  drawCanvas()
}

function updateBlurAreaPosition(e) {
  blurAreaPosition.value = e.detail.value
  drawCanvas()
}

function saveCanvas() {
  if (!canvasContext.value) {
    uni.showToast({ title: 'Canvas未准备好', icon: 'none' })
    return
  }
  uni.showLoading({ title: '保存中...', mask: true })

  uni.canvasToTempFilePath({
    canvasId: canvasId,
    success: (res) => {
      uni.saveImageToPhotosAlbum({
        filePath: res.tempFilePath,
        success: () => {
          uni.hideLoading()
          uni.showToast({ title: '图片已保存到相册', icon: 'success' })
        },
        fail: (err) => {
          uni.hideLoading()
          console.error('保存到相册失败:', err)
          let errMsg = '保存失败'
          if (err.errMsg && err.errMsg.includes('auth')) {
            errMsg = '请授权相册访问权限'
          }
          uni.showToast({ title: errMsg, icon: 'none' })
        }
      })
    },
    fail: (err) => {
      uni.hideLoading()
      console.error('生成临时文件失败:', err)
      uni.showToast({ title: '生成图片失败', icon: 'none' })
    }
  })
}

function goBack() {
  uni.navigateBack()
}
</script>

<style>
.preview-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #e5e5d9; /* 统一背景色 */
  min-height: 100vh;
  padding: 20rpx;
  box-sizing: border-box;
}

.preview-canvas {
  background: #fff;
  margin: 20rpx 0;
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.1);
  border-radius: 8rpx;
}

.slider-container {
  width: 90%;
  margin: 20rpx 0;
  padding: 15rpx;
  background: #fff;
  border-radius: 12rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.1);
}

.slider-container text {
  display: block;
  margin-bottom: 10rpx;
  font-size: 28rpx;
  color: #333;
}

.button-group {
  display: flex;
  justify-content: center;
  width: 100%;
  margin-top: 30rpx;
  gap: 30rpx;
}

.btn {
  font-size: 26rpx;
  padding: 12rpx 30rpx;
  background-color: #c5efd9;
  border-radius: 60rpx;
  color: #333;
  border: 1rpx solid #ccc;
  transition: all 0.2s ease;
}

.btn:active {
  background-color: #c8e6c9;
}

.loading-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  z-index: 999;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.loading-spinner {
  width: 80rpx;
  height: 80rpx;
  border: 8rpx solid rgba(255, 255, 255, 0.3);
  border-top: 8rpx solid #1E90FF;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-text {
  margin-top: 30rpx;
  color: #fff;
  font-size: 30rpx;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
