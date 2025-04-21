<template>
    <view class="container">
      <!-- 微信小程序使用原生camera组件 -->
      <!-- #ifdef MP-WEIXIN -->
      <camera 
        device-position="back" 
        flash="off" 
        style="width: 100vw; height: 100vh;"
        v-if="showCamera"
      ></camera>
      <!-- #endif -->
      
      <!-- App端使用占位视图（实际调用系统相机） -->
      <!-- #ifdef APP-PLUS -->
      <view 
        style="width: 100vw; height: 100vh; background-color: #000;"
        v-if="showCamera"
      ></view>
      <!-- #endif -->
      
      <!-- H5端显示提示 -->
      <!-- #ifdef H5 -->
      <view 
        class="h5-tip"
        v-if="showCamera"
      >
        <text>H5环境请使用手机访问或切换至小程序/App</text>
      </view>
      <!-- #endif -->
      
      <view class="controls">
        <button @click="takePhoto" class="btn">📸 拍照</button>
        <!-- 仅App端需要的切换相机按钮 -->
        <!-- #ifdef APP-PLUS -->
        <button @click="switchCamera" class="btn switch-btn">🔄 切换摄像头</button>
        <!-- #endif -->
      </view>
    </view>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue'
  
  const showCamera = ref(true)
  const cameraContext = ref(null)
  const currentCamera = ref('back') // 仅App端使用
  
  onMounted(() => {
    // #ifdef MP-WEIXIN
    // 微信小程序初始化相机上下文
    setTimeout(() => {
      cameraContext.value = uni.createCameraContext()
      console.log('微信小程序相机初始化完成')
    }, 300)
    // #endif
    
    // #ifdef APP-PLUS
    // App端检查相机权限（修正后的权限检查方式）
    checkAppCameraPermission()
    // #endif
  })
  
  // 修正后的App端相机权限检查
  function checkAppCameraPermission() {
    // #ifdef APP-PLUS
    // 正确的方式是使用plus.gallery和plus.camera，但实际权限检查需要这样处理：
    plus.android.requestPermissions(
      ['android.permission.CAMERA'], // 直接使用权限字符串
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
  
  // 拍照功能
  function takePhoto() {
    // 微信小程序方案
    // #ifdef MP-WEIXIN
    if (!cameraContext.value) {
      uni.showToast({ title: '相机初始化中，请稍候...', icon: 'none' })
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
        console.error('拍照失败:', err)
        uni.showToast({ title: '拍照失败，请重试', icon: 'none' })
      }
    })
    // #endif
    
    // App端方案 - 调用系统相机
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
          uni.showToast({ title: '拍照失败: ' + err.errMsg, icon: 'none' })
        }
      }
    })
    // #endif
    
    // H5端方案
    // #ifdef H5
    uni.showToast({ 
      title: 'H5端请使用手机扫码体验完整功能', 
      icon: 'none',
      duration: 3000
    })
    // #endif
  }
  
  // App端切换摄像头
  function switchCamera() {
    // #ifdef APP-PLUS
    currentCamera.value = currentCamera.value === 'back' ? 'front' : 'back'
    uni.showToast({
      title: `已切换至${currentCamera.value === 'back' ? '后置' : '前置'}摄像头`,
      icon: 'none'
    })
    // #endif
  }
  
  // 跳转到预览页
  function navigateToPreview(imagePath) {
    uni.navigateTo({
      url: `/pages/preview/preview?img=${encodeURIComponent(imagePath)}`,
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
  /* 保持原有样式不变 */
  .container {
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }
  
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
    padding: 12px 24px;
    background-color: rgba(255, 255, 255, 0.7);
    border-radius: 100px;
    font-size: 18px;
    backdrop-filter: blur(10px);
    border: none;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  }
  
  .switch-btn {
    padding: 12px 15px;
  }
  
  /* H5提示样式 */
  /* #ifdef H5 */
  .h5-tip {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #f0f0f0;
    color: #666;
    font-size: 16px;
    text-align: center;
    padding: 20px;
  }
  /* #endif */
  </style>