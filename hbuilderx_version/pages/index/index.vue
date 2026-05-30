<template>
  <scroll-view class="index-page" scroll-y>
    <view class="header">
      <view class="header-content">
        <text class="title">🎨 拼豆小帮手</text>
        <text class="subtitle">把图片变成拼豆艺术</text>
      </view>
      <view class="header-decoration">
        <view class="bean bean-1"></view>
        <view class="bean bean-2"></view>
        <view class="bean bean-3"></view>
        <view class="bean bean-4"></view>
        <view class="bean bean-5"></view>
      </view>
    </view>

    <view class="upload-section">
      <view class="upload-card" @click="chooseImage">
        <view v-if="selectedImage" class="image-preview">
          <image :src="selectedImage" mode="aspectFill" class="preview-image"></image>
          <view class="image-overlay">
            <text class="overlay-text">点击更换图片</text>
          </view>
        </view>
        <view v-else class="upload-placeholder">
          <view class="placeholder-icon">
            <text>📷</text>
          </view>
          <text class="placeholder-title">上传图片</text>
          <text class="placeholder-desc">选择一张图片开始创作</text>
          <view class="upload-options">
            <view class="option-item">
              <text class="option-icon">🖼️</text>
              <text class="option-text">相册</text>
            </view>
          </view>
        </view>
      </view>

      <button
        v-if="selectedImage"
        type="primary"
        @click="generatePixelArt"
        class="btn-generate"
        :loading="generating"
      >
        <text>{{ generating ? '正在生成...' : '✨ 开始创作' }}</text>
      </button>
    </view>

    <view class="features-section">
      <text class="section-title">功能特点</text>
      <view class="feature-list">
        <view class="feature-card">
          <view class="feature-icon-wrap">🎯</view>
          <view class="feature-info">
            <text class="feature-name">智能转换</text>
            <text class="feature-desc">AI算法智能识别颜色</text>
          </view>
        </view>
        <view class="feature-card">
          <view class="feature-icon-wrap">🎨</view>
          <view class="feature-info">
            <text class="feature-name">丰富配色</text>
            <text class="feature-desc">MARD 221色专业色卡</text>
          </view>
        </view>
        <view class="feature-card">
          <view class="feature-icon-wrap">📊</view>
          <view class="feature-info">
            <text class="feature-name">材料清单</text>
            <text class="feature-desc">自动统计所需豆子数量</text>
          </view>
        </view>
        <view class="feature-card">
          <view class="feature-icon-wrap">💾</view>
          <view class="feature-info">
            <text class="feature-name">高清导出</text>
            <text class="feature-desc">支持多尺寸导出</text>
          </view>
        </view>
      </view>
    </view>

    <view class="tips-section">
      <view class="tips-card">
        <text class="tips-icon">💡</text>
        <text class="tips-text">小贴士：建议选择色彩丰富、对比度高的图片，效果更佳！</text>
      </view>
    </view>

    <view class="footer">
      <text class="footer-text">拼豆小帮手 - 让拼豆更简单</text>
    </view>

    <canvas canvas-id="colorCanvas" id="colorCanvas" class="hidden-canvas"></canvas>
  </scroll-view>
</template>

<script>
export default {
  data() {
    return {
      selectedImage: null,
      imageData: null,
      generating: false,
      extractedColors: []
    }
  },
  methods: {
    chooseImage() {
      uni.chooseImage({
        count: 1,
        sizeType: ['compressed'],
        sourceType: ['album'],
        success: (res) => {
          if (res.tempFilePaths && res.tempFilePaths.length > 0) {
            this.selectedImage = res.tempFilePaths[0]
          }
        },
        fail: (err) => {
          uni.showToast({
            title: '选择失败',
            icon: 'none'
          })
          console.error(err)
        }
      })
    },
    generatePixelArt() {
      if (!this.selectedImage) {
        uni.showToast({
          title: '请先选择图片',
          icon: 'none'
        })
        return
      }

      this.generating = true

      uni.navigateTo({
        url: `/pages/preview/preview?imageData=${encodeURIComponent(JSON.stringify({
          tempFilePath: this.selectedImage
        }))}`,
        fail: () => {
          uni.showToast({
            title: '跳转失败',
            icon: 'none'
          })
          this.generating = false
        }
      })

      setTimeout(() => {
        this.generating = false
      }, 1000)
    }
  }
}
</script>

<style scoped>
.index-page {
  height: 100vh;
  background: linear-gradient(135deg, #fff0f6 0%, #fce7f3 25%, #f3e8ff 50%, #e0e7ff 75%, #dbeafe 100%);
  position: relative;
}

.index-page::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: 
    radial-gradient(circle at 20% 20%, rgba(251, 191, 36, 0.15) 0%, transparent 50%),
    radial-gradient(circle at 80% 80%, rgba(168, 85, 247, 0.15) 0%, transparent 50%),
    radial-gradient(circle at 40% 60%, rgba(59, 130, 246, 0.1) 0%, transparent 40%),
    radial-gradient(circle at 60% 20%, rgba(236, 72, 153, 0.1) 0%, transparent 40%);
  pointer-events: none;
}

.header {
  position: relative;
  background: linear-gradient(135deg, #f472b6 0%, #ec4899 30%, #d946ef 60%, #a78bfa 100%);
  padding: calc(40rpx + env(safe-area-inset-top)) 40rpx 140rpx;
  overflow: hidden;
  border-radius: 0 0 48rpx 48rpx;
  box-shadow: 0 8rpx 32rpx rgba(236, 72, 153, 0.3);
}

.header-content {
  position: relative;
  z-index: 1;
  text-align: center;
}

.title {
  font-size: 64rpx;
  font-weight: 800;
  color: #ffffff;
  display: block;
  margin-bottom: 16rpx;
  text-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.25);
  letter-spacing: 4rpx;
}

.subtitle {
  font-size: 32rpx;
  color: rgba(255, 255, 255, 0.95);
  font-weight: 500;
}

.header-decoration {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
}

.bean {
  position: absolute;
  width: 30rpx;
  height: 30rpx;
  border-radius: 50%;
  opacity: 0.3;
  animation: float 4s ease-in-out infinite;
}

.bean-1 {
  background: #fbbf24;
  top: 20%;
  left: 10%;
  animation-delay: 0s;
}

.bean-2 {
  background: #34d399;
  top: 40%;
  right: 15%;
  animation-delay: 1s;
}

.bean-3 {
  background: #60a5fa;
  bottom: 30%;
  left: 20%;
  animation-delay: 2s;
}

.bean-4 {
  background: #f472b6;
  top: 60%;
  right: 25%;
  animation-delay: 0.5s;
}

.bean-5 {
  background: #a78bfa;
  bottom: 20%;
  right: 10%;
  animation-delay: 1.5s;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20rpx); }
}

.upload-section {
  padding: 40rpx;
  margin-top: -60rpx;
}

.upload-card {
  background: #ffffff;
  border-radius: 28rpx;
  overflow: hidden;
  box-shadow: 0 12rpx 40rpx rgba(0, 0, 0, 0.1);
  margin-bottom: 30rpx;
}

.image-preview {
  position: relative;
  height: 420rpx;
}

.preview-image {
  width: 100%;
  height: 100%;
}

.image-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.6));
  padding: 40rpx 30rpx 30rpx;
}

.overlay-text {
  font-size: 26rpx;
  color: #ffffff;
}

.upload-placeholder {
  height: 420rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40rpx;
}

.placeholder-icon {
  width: 120rpx;
  height: 120rpx;
  background: linear-gradient(135deg, #fdf2f8 0%, #fce7f3 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 56rpx;
  margin-bottom: 24rpx;
}

.placeholder-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 12rpx;
}

.placeholder-desc {
  font-size: 26rpx;
  color: #999;
  margin-bottom: 30rpx;
}

.upload-options {
  display: flex;
  align-items: center;
  gap: 30rpx;
}

.option-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8rpx;
}

.option-icon {
  font-size: 40rpx;
}

.option-text {
  font-size: 24rpx;
  color: #666;
}

.option-divider {
  color: #ddd;
  font-size: 24rpx;
}

.btn-generate {
  width: 100%;
  height: 96rpx;
  line-height: 96rpx;
  border-radius: 48rpx;
  font-size: 34rpx;
  font-weight: bold;
  background: linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%);
  color: #ffffff;
  box-shadow: 0 8rpx 24rpx rgba(236, 72, 153, 0.4);
}

.features-section {
  padding: 0 40rpx;
  margin-bottom: 30rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 24rpx;
  padding-left: 10rpx;
}

.feature-list {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20rpx;
}

.feature-card {
  background: #ffffff;
  border-radius: 20rpx;
  padding: 28rpx;
  display: flex;
  align-items: center;
  gap: 20rpx;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.04);
}

.feature-icon-wrap {
  width: 80rpx;
  height: 80rpx;
  background: linear-gradient(135deg, #fdf2f8 0%, #fce7f3 100%);
  border-radius: 20rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 36rpx;
}

.feature-info {
  flex: 1;
}

.feature-name {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 6rpx;
}

.feature-desc {
  font-size: 22rpx;
  color: #999;
}

.tips-section {
  padding: 0 40rpx;
  margin-bottom: 30rpx;
}

.tips-card {
  background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
  border-radius: 20rpx;
  padding: 28rpx;
  display: flex;
  align-items: flex-start;
  gap: 16rpx;
}

.tips-icon {
  font-size: 32rpx;
}

.tips-text {
  font-size: 26rpx;
  color: #92400e;
  line-height: 1.5;
}

.footer {
  padding: 40rpx;
  padding-bottom: calc(40rpx + env(safe-area-inset-bottom));
  text-align: center;
}

.footer-text {
  font-size: 24rpx;
  color: #ccc;
}

.hidden-canvas {
  width: 1rpx;
  height: 1rpx;
  position: absolute;
  top: -9999rpx;
  left: -9999rpx;
}
</style>
