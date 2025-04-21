<template>
    <view class="container">
      <view class="header">设置</view>
  
      <view class="setting-item">
        <text class="label">保存照片到相册</text>
        <switch :checked="saveToAlbum" @change="onToggleSaveToAlbum" />
      </view>
  
      <view class="setting-item">
        <text class="label">使用前置摄像头</text>
        <switch :checked="useFrontCamera" @change="onToggleCameraDirection" />
      </view>
    </view>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  
  const saveToAlbum = ref(true)
  const useFrontCamera = ref(false)
  
  function onToggleSaveToAlbum(e) {
    saveToAlbum.value = e.detail.value
    uni.showToast({ title: `已${saveToAlbum.value ? '启用' : '关闭'}保存到相册`, icon: 'none' })
  }
  
  function onToggleCameraDirection(e) {
    useFrontCamera.value = e.detail.value
    uni.showToast({ title: `已切换为${useFrontCamera.value ? '前置' : '后置'}摄像头`, icon: 'none' })
  }
  </script>
  
  <style>
  .container {
    padding: 40rpx;
    background-color: #f5f5f5;
    height: 100vh;
  }
  
  .header {
    font-size: 36rpx;
    font-weight: bold;
    margin-bottom: 40rpx;
    color: #333;
  }
  
  .setting-item {
    background-color: #fff;
    padding: 30rpx;
    border-radius: 20rpx;
    margin-bottom: 20rpx;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .label {
    font-size: 30rpx;
    color: #444;
  }
  </style>
  