<template>
  <view class="preview-container">
    <image :src="imgPath" :mode="imgMode" class="preview-img" @load="handleImageLoad" />

    <view class="button-group">
      <button class="btn save-btn" @click="saveImage">💾 保存照片</button>
      <button class="btn back-btn" @click="goBack">🔙 返回首页</button>
    </view>
  </view>
</template>

<script setup>
import { onLoad } from '@dcloudio/uni-app'
import { ref } from 'vue'

const imgPath = ref('')
const imgMode = ref('aspectFill')

onLoad((options) => {
  imgPath.value = decodeURIComponent(options.img || '')
})

function handleImageLoad(e) {
  uni.getImageInfo({
    src: imgPath.value,
    success: (res) => {
      imgMode.value = res.width > res.height ? 'aspectFit' : 'aspectFill'
    }
  })
}

function saveImage() {
  uni.saveImageToPhotosAlbum({
    filePath: imgPath.value,
    success: () => {
      uni.showToast({ title: '已保存', icon: 'success' })
    },
    fail: () => {
      uni.showToast({ title: '保存失败', icon: 'none' })
    }
  })
}

function goBack() {
  uni.reLaunch({
    url: '/pages/index'
  })
}
</script>

<style>
.preview-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #555555;
  height: 100vh;
  padding: 30rpx;
}

.preview-img {
  width: 100%;
  height: 75vh;
  object-fit: cover;
  margin-bottom: 30rpx;
}

.button-group {
  display: flex;
  gap: 30rpx;
}

.btn {
  padding: 20rpx 50rpx;
  font-size: 30rpx;
  border-radius: 100rpx;
  background: #ffffff;
  box-shadow: 0 6rpx 12rpx rgba(0, 0, 0, 0.1);
  color: #333;
}

.save-btn {
  background: linear-gradient(to right, #1E90FF, #0066CC);
  color: white;
  opacity: 0.8;
}

.back-btn {
  background: linear-gradient(to right, #FFA500, #FF4500);
  color: white;
  opacity: 0.8;
}
</style>
