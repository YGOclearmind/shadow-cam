<template>
  <view class="preview-container">
    <!-- 图片预览 -->
    <image 
      v-if="mode !== 'record'"
      :src="imgPath" 
      :mode="imgMode" 
      class="preview-img" 
      @load="handleImageLoad" 
    />
    
    <!-- 视频预览 -->
    <video
      v-if="mode === 'record'"
      :src="imgPath"
      class="preview-video"
      controls
      autoplay
      loop
    ></video>

    <view class="button-group">
      <button class="btn save-btn" @click="saveMedia">💾 {{ mode === 'record' ? '保存视频' : '保存照片' }}</button>
      <button class="btn back-btn" @click="goBack">🔙 返回首页</button>
    </view>
  </view>
</template>

<script setup>
import { onLoad } from '@dcloudio/uni-app'
import { ref } from 'vue'

const imgPath = ref('')
const imgMode = ref('aspectFill')
const mode = ref('')

onLoad((options) => {
  imgPath.value = decodeURIComponent(options.img || '')
  mode.value = options.mode || ''
})

function handleImageLoad(e) {
  uni.getImageInfo({
    src: imgPath.value,
    success: (res) => {
      imgMode.value = res.width > res.height ? 'aspectFit' : 'aspectFill'
    }
  })
}

function saveMedia() {
  if (mode.value === 'record') {
    // 保存视频
    uni.saveVideoToPhotosAlbum({
      filePath: imgPath.value,
      success: () => {
        uni.showToast({ title: '视频已保存', icon: 'success' })
      },
      fail: () => {
        uni.showToast({ title: '视频保存失败', icon: 'none' })
      }
    })
  } else {
    // 保存图片
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

.preview-container {
  display: flex;
  flex-direction: column;
  justify-content: space-between; /* 均匀分布空间 */
  align-items: center;
  background: #e5e5d9;
  height: 100vh;
  padding: 20rpx;
  box-sizing: border-box;
}

.preview-img, .preview-video {
  width: 100%;
  height: 60vh; /* 进一步减少高度 */
  object-fit: contain;
  margin-bottom: 20rpx;
  overflow: hidden;
}

.preview-video {
  background-color: #000;
  height: 55vh; /* 视频高度更小 */
}

.button-group {
  display: flex;
  gap: 30rpx;
  min-height: 100rpx; /* 确保按钮区域高度 */
  align-items: center;
  width: 100%;
  justify-content: center;
  padding: 20rpx 0;
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
