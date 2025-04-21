<template>
    <view class="preview-container">
      <image :src="imgPath" mode="widthFix" class="preview-img" />
      <view class="actions">
        <button @click="back">返回重拍</button>
      </view>
    </view>
  </template>
  
  <script setup>
  import { ref } from 'vue'
  import { onLoad } from '@dcloudio/uni-app'
  const imgPath = ref('')
  
  onLoad((options) => {
    imgPath.value = decodeURIComponent(options.img || '')
  })
  
  function back() {
    uni.navigateBack()
  }
  </script>
  
  <style>
  .preview-container {
    padding: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .preview-img {
    max-width: 100%;
    border-radius: 12px;
    margin-bottom: 20px;
  }
  .actions {
    display: flex;
    gap: 10px;
  }
  </style>
  