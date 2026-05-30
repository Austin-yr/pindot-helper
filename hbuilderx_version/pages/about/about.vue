<template>
  <view class="about-page">
    <view class="about-header">
      <view class="app-icon">🎨</view>
      <text class="app-name">像素图生成器</text>
      <text class="app-version">版本 1.0.0</text>
    </view>

    <view class="about-section">
      <text class="section-title">功能特点</text>
      <view class="feature-list">
        <view class="feature-item">
          <text class="feature-icon">📷</text>
          <view class="feature-content">
            <text class="feature-title">图片上传</text>
            <text class="feature-desc">支持相册选择和拍照上传</text>
          </view>
        </view>
        <view class="feature-item">
          <text class="feature-icon">🎯</text>
          <view class="feature-content">
            <text class="feature-title">智能转换</text>
            <text class="feature-desc">自动将图片转换为像素风格</text>
          </view>
        </view>
        <view class="feature-item">
          <text class="feature-icon">🎨</text>
          <view class="feature-content">
            <text class="feature-title">拼豆配色</text>
            <text class="feature-desc">预设20种常用拼豆颜色</text>
          </view>
        </view>
        <view class="feature-item">
          <text class="feature-icon">📐</text>
          <view class="feature-content">
            <text class="feature-title">网格预览</text>
            <text class="feature-desc">显示像素网格方便制作</text>
          </view>
        </view>
      </view>
    </view>

    <view class="about-section">
      <text class="section-title">使用说明</text>
      <view class="instruction">
        <text class="instruction-step">1. 点击上传按钮选择图片</text>
        <text class="instruction-step">2. 等待系统生成像素图</text>
        <text class="instruction-step">3. 调整网格尺寸和显示选项</text>
        <text class="instruction-step">4. 导出图片保存到相册</text>
      </view>
    </view>

    <view class="about-footer">
      <text class="copyright">© 2024 像素图生成器</text>
      <text class="tip">仅供娱乐和学习使用</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {}
  }
}
</script>

<style scoped>
.about-page {
  min-height: 100vh;
  background-color: #f5f5f5;
  padding: 20rpx;
}

.about-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 24rpx;
  padding: 60rpx 40rpx;
  text-align: center;
  margin-bottom: 20rpx;
}

.app-icon {
  font-size: 120rpx;
  margin-bottom: 20rpx;
}

.app-name {
  font-size: 40rpx;
  font-weight: bold;
  color: #ffffff;
  display: block;
  margin-bottom: 10rpx;
}

.app-version {
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.8);
}

.about-section {
  background: #ffffff;
  border-radius: 16rpx;
  padding: 40rpx;
  margin-bottom: 20rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 30rpx;
}

.feature-list {
  display: flex;
  flex-direction: column;
  gap: 30rpx;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 24rpx;
}

.feature-icon {
  font-size: 60rpx;
  width: 80rpx;
  text-align: center;
}

.feature-content {
  flex: 1;
}

.feature-title {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 6rpx;
}

.feature-desc {
  font-size: 24rpx;
  color: #999;
}

.instruction {
  display: flex;
  flex-direction: column;
  gap: 20rpx;
}

.instruction-step {
  font-size: 28rpx;
  color: #666;
  padding-left: 30rpx;
  position: relative;
}

.instruction-step::before {
  content: counter(step);
  counter-increment: step;
  position: absolute;
  left: 0;
  top: 0;
  width: 40rpx;
  height: 40rpx;
  background: #667eea;
  color: #ffffff;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24rpx;
  font-weight: bold;
}

.about-section:nth-child(2) .instruction {
  counter-reset: step;
}

.about-footer {
  text-align: center;
  padding: 40rpx 0;
}

.copyright {
  font-size: 24rpx;
  color: #999;
  display: block;
  margin-bottom: 10rpx;
}

.tip {
  font-size: 22rpx;
  color: #ccc;
}
</style>
