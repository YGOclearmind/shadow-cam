<template>
  <view class="preview-container">
    <image :src="imgPath" mode="aspectFit" class="preview-img" />

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

onLoad((options) => {
  imgPath.value = decodeURIComponent(options.img || '')
})

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
  background: #f5f7fa;
  height: 100vh;
  padding: 30rpx;
}

.preview-img {
  width: 90%;
  max-height: 70vh;
  border-radius: 20rpx;
  box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, 0.1);
  margin-bottom: 50rpx;
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
  background: linear-gradient(to right, #4facfe, #00f2fe);
  color: white;
}

.back-btn {
  background: linear-gradient(to right, #fbc2eb, #a6c1ee);
  color: white;
}
</style>
