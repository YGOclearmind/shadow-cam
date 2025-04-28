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
  background: #e5e5d9; /* 统一背景色 */
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
</style>
