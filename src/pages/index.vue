<template>
  <view class="container">
    <view class="logo-area">
      <image class="logo" src="/static/ShadowCam.png" mode="aspectFit" />
      <text class="title">ShadowCam</text>
      <text class="subtitle">无形自拍机，记录不留痕</text>
    </view>
    <view class="actions">
      <button class="start-btn" @click="goToCamera">📸 开始拍摄</button>
    </view>
    <view class="secondary-actions">
      <button class="mini-btn" @click="goToSettings">⚙️ 设置</button>
      <button class="mini-btn" @click="goToAbout">ℹ️ 关于</button>
    </view>
  </view>
</template>

<script setup>
function goToCamera() {
uni.navigateTo({ url: '/pages/camera' })
}
function goToSettings() {
uni.navigateTo({ url: '/pages/settings' })
}
function goToAbout() {
uni.navigateTo({ url: '/pages/about' })
}
</script>

<style>
.container {
display: flex;
flex-direction: column;
align-items: center;
justify-content: space-between;
height: 90vh;
padding: 80rpx 30rpx 100rpx;
background: linear-gradient(to bottom, #fdfbfb, #ebedee);
}

.logo-area {
display: flex;
flex-direction: column;
align-items: center;
margin-top: 100rpx;
}

.logo {
width: 180rpx;
height: 180rpx;
margin-bottom: 30rpx;
border-radius: 12px;
}

.title {
font-size: 40rpx;
font-weight: bold;
color: #333;
margin-bottom: 10rpx;
}

.subtitle {
font-size: 26rpx;
color: #888;
}

/* 调整开始拍摄按钮的外边距 */
.actions {
margin-top: 25rpx;
}

.start-btn {
background-color: #a0d8a0;
color: white;
font-size: 32rpx;
padding: 20rpx 60rpx;
border-radius: 100rpx;
box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.1);
transition: all 0.2s ease;
}

.start-btn:active {
background-color: #88c088;
}

/* 增加设置 / 关于 按钮组的外边距，使间距更合理 */
.secondary-actions {
display: flex;
justify-content: center;
gap: 30rpx;
margin-top: 15rpx; /* 增加此行代码 */
}

.mini-btn {
font-size: 26rpx;
padding: 12rpx 30rpx;
background-color: #ffffffcc;
border-radius: 60rpx;
color: #333;
border: 1rpx solid #ccc;
}
</style>
