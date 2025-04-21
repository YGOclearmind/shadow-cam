<template>
    <view class="container">
      <view class="logo-area">
        <image class="logo" src="/static/logo.png" mode="aspectFit" />
        <text class="title">ShadowCam</text>
        <text class="version">版本 1.0.0</text>
      </view>
  
      <view class="info">
        <text class="paragraph">
          ShadowCam 是一款注重隐私与疗愈体验的创意自拍工具。拍照后不留下历史记录，给予你沉浸、轻松、不被打扰的影像空间。
        </text>
        <text class="paragraph">
          项目由创新实践小组开发，基于 Vue3 + UniApp 实现跨平台体验。
        </text>
      </view>
    </view>
  </template>
  
  <script setup>
  </script>
  
  <style>
  .container {
    padding: 40rpx;
    background: linear-gradient(to bottom, #f8f9fa, #e8eaf0);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .logo-area {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 80rpx;
    margin-bottom: 60rpx;
  }
  
  .logo {
    width: 160rpx;
    height: 160rpx;
    margin-bottom: 20rpx;
  }
  
  .title {
    font-size: 38rpx;
    font-weight: bold;
    color: #333;
  }
  
  .version {
    font-size: 26rpx;
    color: #888;
    margin-top: 10rpx;
  }
  
  .info {
    padding: 20rpx 30rpx;
  }
  
  .paragraph {
    font-size: 28rpx;
    line-height: 44rpx;
    color: #555;
    margin-bottom: 30rpx;
  }
  </style>
  