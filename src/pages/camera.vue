<template>
  <view class="container">
    <!-- 微信小程序使用原生 camera 组件 -->
    <!-- #ifdef MP-WEIXIN -->
    <camera 
      class="camera-view"
      :device-position="currentCamera"
      :flash="flashMode" 
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

    <!-- 顶部控制栏 -->
    <view class="top-controls">
      <!-- #ifdef APP-PLUS || MP-WEIXIN -->
      <view class="control-btn" @click="switchCamera">
        <uni-icons type="reload" size="24" color="#fff"></uni-icons>
      </view>
      <!-- #endif -->
      <view class="control-btn" @click="toggleFlash">
        <uni-icons :type="flashIcon" size="24" color="#fff"></uni-icons>
      </view>
    </view>

    <!-- 模式切换栏 -->
    <scroll-view
      class="mode-switch"
      scroll-x
      :scroll-left="scrollLeft"
      show-scrollbar="false"
    >
      <view
        v-for="mode in modes"
        :key="mode.value"
        :class="['mode-item', currentMode === mode.value ? 'active' : '']"
        @click="changeMode(mode.value)"
      >
        <view class="mode-icon">
          <uni-icons :type="mode.icon" size="18" :color="currentMode === mode.value ? '#fff' : '#333'"></uni-icons>
        </view>
        {{ mode.label }}
      </view>
    </scroll-view>

    <!-- 拍照按钮区域 -->
    <view class="controls">
      <view class="capture-btn" @click="takePhoto">
        <view class="capture-inner"></view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import uniIcons from '@dcloudio/uni-ui/lib/uni-icons/uni-icons.vue'

const showCamera = ref(true)
const cameraContext = ref(null)
const currentCamera = ref('back')
const currentMode = ref('silent')
const scrollLeft = ref(0)
const flashMode = ref('off')
const flashIcon = ref('flash-off')

const modes = [
  { label: '静默', value: 'silent', icon: 'eye' },
  { label: '计时', value: 'timed', icon: 'eye' },
  { label: '模糊', value: 'blur', icon: 'image' },
  { label: '遮挡', value: 'mask', icon: 'person' },
  { label: '记录', value: 'record', icon: 'videocam' },
  { label: '伪装', value: 'disguise', icon: 'eye' },
]

const toggleFlash = () => {
  const modes = ['off', 'on', 'auto']
  const icons = ['flash-off', 'flash-on', 'flash-auto']
  const currentIndex = modes.indexOf(flashMode.value)
  const nextIndex = (currentIndex + 1) % modes.length
  
  flashMode.value = modes[nextIndex]
  flashIcon.value = icons[nextIndex]
  
  // 实际切换闪光灯逻辑
  // #ifdef MP-WEIXIN
  if (cameraContext.value) {
    cameraContext.value.setFlashMode({
      mode: flashMode.value,
      success: () => console.log('闪光灯模式切换成功:', flashMode.value),
      fail: (err) => console.error('闪光灯模式切换失败:', err)
    })
  }
  // #endif
}

const changeMode = (mode) => {
  currentMode.value = mode
}

const switchCamera = () => {
  currentCamera.value = currentCamera.value === 'back' ? 'front' : 'back'
  uni.showToast({
    title: `切换至${currentCamera.value === 'back' ? '后置' : '前置'}摄像头`,
    icon: 'none'
  })
}

const takePhoto = () => {
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
      navigateToPreview(res.tempFilePaths[0])
    },
    fail: (err) => {
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

const navigateToPreview = (imagePath) => {
  uni.navigateTo({
    url: `/pages/preview?img=${encodeURIComponent(imagePath)}`,
    fail: (err) => {
      uni.previewImage({
        urls: [imagePath],
        current: 0
      })
    }
  })
}

const showPermissionDeniedAlert = () => {
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

onMounted(() => {
  // #ifdef MP-WEIXIN
  setTimeout(() => {
    cameraContext.value = uni.createCameraContext()
  }, 300)
  // #endif

  // #ifdef APP-PLUS
  checkAppCameraPermission()
  // #endif
})

const checkAppCameraPermission = () => {
  // #ifdef APP-PLUS
  plus.android.requestPermissions(
    ['android.permission.CAMERA'],
    (e) => {
      if (e.deniedAlways && e.deniedAlways.length > 0) {
        showPermissionDeniedAlert()
      }
    }
  )
  // #endif
}
</script>

<style>
.container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: #000;
}

.camera-view {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
}

.top-controls {
  position: absolute;
  top: 40px;
  right: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  z-index: 20;
}

.top-controls .control-btn {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  transition: all 0.2s ease;
}

.top-controls .control-btn:active {
  transform: scale(0.95);
  background: rgba(0, 0, 0, 0.5);
}

.controls {
  position: absolute;
  bottom: 60px;
  width: 100%;
  display: flex;
  justify-content: center;
  z-index: 10;
}

.controls .capture-btn {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  border: 4px solid rgba(255, 255, 255, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.controls .capture-btn .capture-inner {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: #fff;
  transition: all 0.2s ease;
}

.controls .capture-btn:active {
  transform: scale(0.95);
  border-color: rgba(255, 255, 255, 0.8);
}

.controls .capture-btn:active .capture-inner {
  transform: scale(0.9);
  background: rgba(255, 255, 255, 0.8);
}

.h5-tip {
  width: 100%;
  height: 100%;
  background-color: #1a1a1a;
  color: #ccc;
  font-size: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px;
  text-align: center;
  line-height: 1.6;
}

.mode-switch {
  position: absolute;
  bottom: 160px;
  left: 0;
  width: 100%;
  padding: 12px 16px;
  display: flex;
  flex-direction: row;
  gap: 12px;
  z-index: 20;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(20px);
  box-shadow: 0 2px 15px rgba(0, 0, 0, 0.2);
}

.mode-switch .mode-item {
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 20px;
  font-size: 14px;
  color: #333;
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: all 0.2s ease;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.mode-switch .mode-item .mode-icon {
  display: flex;
  align-items: center;
}

.mode-switch .mode-item.active {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  color: #fff;
  font-weight: 500;
  box-shadow: 0 4px 10px rgba(0, 178, 255, 0.3);
}

@media (max-height: 700px) {
  .mode-switch {
    bottom: 140px;
    padding: 10px 12px;
  }
  
  .mode-switch .mode-item {
    padding: 6px 12px;
    font-size: 13px;
  }
  
  .controls {
    bottom: 40px;
  }
}
</style>