<template>
  <view class="container">
    <!-- 微信小程序使用原生 camera 组件 -->
    <!-- #ifdef MP-WEIXIN -->
    <camera 
      class="camera-view"
      device-position="back" 
      flash="off" 
      v-if="showCamera"
    ></camera>
    <!-- #endif -->

    <!-- App端用黑色占位层表示系统相机 -->
    <!-- #ifdef APP-PLUS -->
    <view 
      class="camera-view"
      v-if="showCamera"
    ></view>
    <!-- #endif -->

    <!-- H5 端提示 -->
    <!-- #ifdef H5 -->
    <view class="h5-tip" v-if="showCamera">
      <text>H5环境请使用手机访问或切换至小程序/App</text>
    </view>
    <!-- #endif -->

    <!-- 拍照按钮 -->
    <view class="controls">
      <button @click="takePhoto" class="btn">📸 拍照</button>
      <!-- #ifdef APP-PLUS -->
      <button @click="switchCamera" class="btn switch-btn">🔄 切换</button>
      <!-- #endif -->
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const showCamera = ref(true)
const cameraContext = ref(null)
const currentCamera = ref('back')

onMounted(() => {
  // #ifdef MP-WEIXIN
  setTimeout(() => {
    cameraContext.value = uni.createCameraContext()
    console.log('微信小程序相机初始化完成')
  }, 300)
  // #endif

  // #ifdef APP-PLUS
  checkAppCameraPermission()
  // #endif
})

function checkAppCameraPermission() {
  // #ifdef APP-PLUS
  plus.android.requestPermissions(
    ['android.permission.CAMERA'],
    (e) => {
      console.log('申请相机权限结果:', e)
      if (e.deniedAlways && e.deniedAlways.length > 0) {
        showPermissionDeniedAlert()
      }
    },
    (e) => {
      console.log('权限检查错误:', e)
    }
  )
  // #endif
}

function showPermissionDeniedAlert() {
  uni.showModal({
    title: '权限提示',
    content: '您拒绝了相机权限，将无法使用拍照功能',
    confirmText: '去设置',
    success: (res) => {
      if (res.confirm) {
        // #ifdef APP-PLUS
        plus.runtime.openURL(plus.android.getProperty('app', 'packageName'))
        // #endif
      }
    }
  })
  showCamera.value = false
}

function takePhoto() {
  // #ifdef MP-WEIXIN
  if (!cameraContext.value) {
    uni.showToast({ title: '相机初始化中...', icon: 'none' })
    return
  }
  uni.showLoading({ title: '拍摄中...', mask: true })
  cameraContext.value.takePhoto({
    quality: 'high',
    success: (res) => {
      uni.hideLoading()
      console.log('拍照成功:', res)
      navigateToPreview(res.tempImagePath)
    },
    fail: (err) => {
      uni.hideLoading()
      uni.showToast({ title: '拍照失败', icon: 'none' })
      console.error('拍照失败:', err)
    }
  })
  // #endif

  // #ifdef APP-PLUS
  uni.chooseImage({
    sourceType: ['camera'],
    camera: currentCamera.value === 'front' ? 'front' : 'back',
    success: (res) => {
      console.log('拍照成功:', res)
      navigateToPreview(res.tempFilePaths[0])
    },
    fail: (err) => {
      console.error('拍照失败:', err)
      if (err.errMsg.includes('permission')) {
        showPermissionDeniedAlert()
      } else {
        uni.showToast({ title: '拍照失败', icon: 'none' })
      }
    }
  })
  // #endif

  // #ifdef H5
  uni.showToast({ 
    title: '请使用手机端体验完整功能', 
    icon: 'none',
    duration: 3000
  })
  // #endif
}

function switchCamera() {
  // #ifdef APP-PLUS
  currentCamera.value = currentCamera.value === 'back' ? 'front' : 'back'
  uni.showToast({
    title: `切换至${currentCamera.value === 'back' ? '后置' : '前置'}摄像头`,
    icon: 'none'
  })
  // #endif
}

function navigateToPreview(imagePath) {
  uni.navigateTo({
    url: `/pages/preview?img=${encodeURIComponent(imagePath)}`,
    fail: (err) => {
      console.error('跳转失败:', err)
      uni.previewImage({
        urls: [imagePath],
        current: 0
      })
    }
  })
}
</script>

<style>
.container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: #333333;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.camera-view {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}

/* 控制按钮区域 */
.controls {
  position: absolute;
  bottom: 40px;
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 20px;
  z-index: 10;
}

.btn {
  padding: 14px 30px;
  border-radius: 100px;
  font-size: 18px;
  color: #000;
  box-shadow: 2px 2px 3px rgba(51, 51, 51, 0.5);
  border: none;
  transition: transform 0.1s ease;
  opacity: 0.8;
}

.btn:first-child {
  background: linear-gradient(to bottom, #90EE90, #008000);
}

.btn:last-child {
  background: linear-gradient(to bottom, #FFC04D, #FF6B00);
}

.btn:active {
  transform: scale(0.96);
}

/* H5提示样式 */
.h5-tip {
  width: 100%;
  height: 100%;
  background-color: #f0f0f0;
  color: #666;
  font-size: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px;
}
</style>
