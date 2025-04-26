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

    <!-- 新增特效画布 -->
    <canvas 
    id="effectCanvas"
    v-show="showEffectCanvas" 
    class="effect-canvas" 
    :width="canvasSize.pixelWidth"
    :height="canvasSize.pixelHeight"
    :style="{ width: canvasSize.width, height: canvasSize.height }"
  ></canvas>

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
    <view class="mode-switch">
      <view
        v-for="mode in modes"
        :key="mode.value"
        :class="['mode-item', currentMode === mode.value ? 'active' : '']"
        @click="changeMode(mode.value)"
      >
        <view class="mode-icon">
          <uni-icons :type="mode.icon" size="24" :color="currentMode === mode.value ? '#fff' : '#333'"></uni-icons>
        </view>
        <view class="mode-label">{{ mode.label }}</view>
      </view>
    </view>

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
import { nextTick } from 'vue'

const showCamera = ref(true)
const cameraContext = ref(null)
const currentCamera = ref('back')
const currentMode = ref('silent')
const scrollLeft = ref(0)
const flashMode = ref('off')
const flashIcon = ref('flash-off')

const canvasSize = ref({
  width: '100vw',
  height: '100vh',
  pixelWidth: 300,
  pixelHeight: 400
})

const modes = [
  { label: '静默模式', value: 'silent', icon: 'eye-slash-filled' }, // 眼睛图标
  { label: '计时拍摄', value: 'timed', icon: 'spinner-cycle' }, // 时钟图标
  { label: '模糊背景', value: 'blur', icon: 'image-filled' }, // 图片图标
  { label: '人脸遮挡', value: 'mask', icon: 'person-filled' }, // 人物图标
  { label: '视频录制', value: 'record', icon: 'videocam' }, // 视频图标
  { label: '伪装模式', value: 'disguise', icon: 'help' } // 帮助图标
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
  console.log('当前模式:', mode, '图标:', modes.find(m => m.value === mode)?.icon)
}

const switchCamera = () => {
  currentCamera.value = currentCamera.value === 'back' ? 'front' : 'back'
  uni.showToast({
    title: `切换至${currentCamera.value === 'back' ? '后置' : '前置'}摄像头`,
    icon: 'none'
  })
}

const takePhoto = () => {
  if (currentMode.value === 'timed') {
    let count = 3
    const timer = setInterval(() => {
      if (count > 0) {
        uni.showToast({
          title: `${count--}`,
          icon: 'none',
          duration: 800
        })
      } else {
        clearInterval(timer)
        performPhoto() // 执行真正的拍照逻辑
      }
    }, 1000)
  } else if (currentMode.value === 'silent') {
    doSilentCapture()
    return
  }
  else {
    performPhoto()
  }
}
const performPhoto = () => {
  // #ifdef MP-WEIXIN
  if (!cameraContext.value) {
    uni.showToast({ title: '相机初始化中...', icon: 'none' })
    return
  }
  uni.showLoading({ title: '拍摄中...', mask: true })
 cameraContext.value.takePhoto({
    success: async (res) => {
      const processed = await processImage(res.tempImagePath)
      navigateToPreview(processed)
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
const doSilentCapture = () => {
  // #ifdef MP-WEIXIN
  if (!cameraContext.value) return
  cameraContext.value.takePhoto({
    quality: 'low', // 静默模式可用 low 提高隐蔽性和速度
    success: (res) => {
      console.log('静默拍照成功:', res.tempImagePath)
      // TODO: 可以把照片保存、上传、加入队列等
    },
    fail: (err) => {
      console.error('静默拍照失败:', err)
    }
  })
  // #endif

  // #ifdef APP-PLUS
  uni.chooseImage({
    sourceType: ['camera'],
    success: (res) => {
      console.log('静默模式 - APP拍照成功:', res.tempFilePaths[0])
    },
    fail: (err) => {
      console.error('静默模式拍照失败:', err)
    }
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
    }
  })
  showCamera.value = false
}

onMounted(async() => {
  // #ifdef MP-WEIXIN
  setTimeout(() => {
    cameraContext.value = uni.createCameraContext()
  }, 300)
  // #endif

  // #ifdef APP-PLUS
  checkAppCameraPermission()
  // #endif
})

/ 获取相机组件尺寸
  await nextTick()
  const query = uni.createSelectorQuery().in(this)
  query.select('.camera-view').boundingClientRect(rect => {
    if (rect) {
      const pixelRatio = uni.getSystemInfoSync().pixelRatio
      canvasSize.value = {
        width: rect.width + 'px',
        height: rect.height + 'px',
        pixelWidth: rect.width * pixelRatio,
        pixelHeight: rect.height * pixelRatio
      }
    }
  }).exec()
})

// 新增图像处理方法
const processImage = async (tempFilePath) => {
  if (currentMode.value === 'blur') {
    return await applyBlurEffect(tempFilePath)
  }
  if (currentMode.value === 'mask') {
    return await applyFaceMask(tempFilePath)
  }
  return tempFilePath
}
// 模糊效果实现
const applyBlurEffect = (tempFilePath) => {
  return new Promise((resolve, reject) => {
    const ctx = uni.createCanvasContext('effectCanvas', this)
    
    // 使用缩放实现简易模糊
    ctx.save()
    ctx.scale(0.3, 0.3)
    ctx.drawImage(tempFilePath, 0, 0, canvasSize.value.pixelWidth / 0.3, canvasSize.value.pixelHeight / 0.3)
    ctx.restore()
    
    ctx.draw(false, () => {
      uni.canvasToTempFilePath({
        canvasId: 'effectCanvas',
        success: (res) => resolve(res.tempFilePath),
        fail: reject
      }, this)
    })
  })
}

// 人脸遮挡实现
const applyFaceMask = async (tempFilePath) => {
  try {
    // 模拟人脸检测结果
    const faceRect = await detectFace(tempFilePath)
    
    const ctx = uni.createCanvasContext('effectCanvas', this)
    ctx.drawImage(tempFilePath, 0, 0, canvasSize.value.pixelWidth, canvasSize.value.pixelHeight)
    
    // 绘制遮挡区域
    ctx.beginPath()
    ctx.arc(faceRect.x + faceRect.width/2, faceRect.y + faceRect.height/2, 
            Math.min(faceRect.width, faceRect.height)/2, 0, Math.PI * 2)
    ctx.fillStyle = 'rgba(0,0,0,0.7)'
    ctx.fill()
    
    return await new Promise((resolve, reject) => {
      ctx.draw(false, () => {
        uni.canvasToTempFilePath({
          canvasId: 'effectCanvas',
          success: (res) => resolve(res.tempFilePath),
          fail: reject
        }, this)
      })
    })
  } catch (err) {
    console.error('人脸遮挡失败:', err)
    return tempFilePath
  }
}

// 模拟人脸检测（实际项目需替换真实检测逻辑）
const detectFace = (filePath) => {
  return new Promise((resolve) => {
    // 获取图片信息
    uni.getImageInfo({
      src: filePath,
      success: (res) => {
        // 模拟人脸在图片中央
        resolve({
          x: res.width * 0.25,
          y: res.height * 0.25,
          width: res.width * 0.5,
          height: res.height * 0.5
        })
      }
    })
  })
}

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
// 注册组件
const components = {
  'uni-icons': uniIcons
};
</script>

<style>
.container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: #ffffff; /* 白色背景 */
  color: #000000; /* 黑色文字 */
  font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-weight: 400;
  letter-spacing: 0.5px;
}

/* 新增canvas样式 */
.effect-canvas {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 5;
  pointer-events: none;
}

.camera-view {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 2;
  background: #ffffff; /* 白色背景 */
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
  background: rgba(0, 0, 0, 0.3); /* 深灰色背景 */
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  transition: all 0.2s ease;
}

.top-controls .control-btn:active {
  transform: scale(0.95);
  background: rgba(0, 0, 0, 0.5); /* 更深的灰色背景 */
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
  background: rgba(245, 245, 245, 0.9); /* 浅灰色背景 */
  border: 4px solid rgba(230, 230, 230, 0.9); /* 浅灰色边框 */
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.05); /* 添加内阴影 */
}

.controls .capture-btn .capture-inner {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: rgba(220, 220, 220, 0.9); /* 加深的灰色 */
  transition: all 0.2s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05); /* 添加轻微阴影 */
}

.controls .capture-btn:active {
  transform: scale(0.95);
  border-color: rgba(220, 220, 220, 0.9); /* 点击时加深边框 */
}

.controls .capture-btn:active .capture-inner {
  transform: scale(0.9);
  background: rgba(210, 210, 210, 0.9); /* 点击时加深的灰色 */
}

.h5-tip {
  width: 100%;
  height: 100%;
  background-color: #1a1a1a; /* 更深的灰色背景 */
  color: #ccc; /* 灰色文字 */
  font-size: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px;
  text-align: center;
  line-height: 1.6;
}

@keyframes flash {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.mode-switch {
  position: absolute;
  bottom: 160px;
  left: 50%;
  transform: translateX(-50%);
  width: 90%;
  padding: 12px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
  gap: 15px; /* 增大间距 */
  z-index: 20;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  border-radius: 20px;
}

.mode-switch .mode-item {
  height: 70px; /* 减小高度 */
  background: #e8f5e9;
  border-radius: 12px;
  color: #000000;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.15s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  font-family: 'PingFang SC';
  border: 2px solid transparent;
  will-change: transform;
}

.mode-switch .mode-item .mode-icon {
  height: 60px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  color: #000000; /* 黑色图标 */
}

.mode-switch .mode-item .mode-label {
  height: 20px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #000000; /* 黑色文字 */
}

.mode-switch .mode-item.active {
  transform: scale(1.1);
  background: #c8e6c9;
  animation: flash 0.3s ease; /* 添加闪烁效果 */
  transition: transform 0.2s ease;
}

@media (max-height: 700px) {
  .mode-switch {
    bottom: 140px;
    padding: 10px;
    border-radius: 15px; /* 调整圆角 */
    gap: 8px;
  }
  
  .mode-switch .mode-item {
    height: 70px;
    border-radius: 10px; /* 更小的圆角使其更圆润 */
  }

  .mode-switch .mode-item .mode-icon {
    height: 50px;
    font-size: 20px;
  }

  .mode-switch .mode-item .mode-label {
    font-size: 11px;
  }
  
  .controls {
    bottom: 40px;
  }
}
</style>
