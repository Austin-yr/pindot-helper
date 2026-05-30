<template>
  <view class="preview-page">
    <scroll-view class="preview-content" scroll-y>
      <view class="preview-header">
        <view class="header-top">
          <view class="back-btn" @click="regenerate">
            <text>‹</text>
          </view>
          <text class="title">拼豆小帮手 · MARD 221色</text>
          <view style="width: 80rpx;"></view>
        </view>
        <view class="settings">
          <view class="setting-row full-width">
            <text class="setting-label">图片尺寸</text>
            <view class="size-options">
              <view
                v-for="size in boardSizes"
                :key="size.value"
                :class="['size-option', { active: boardSize === size.value }]"
                @click="selectSize(size.value)"
              >
                <text>{{ size.label }}</text>
              </view>
            </view>
          </view>
          <view class="setting-row">
            <text class="setting-label">行列号</text>
            <switch :checked="showGrid" @change="toggleGrid" color="#ec4899" />
          </view>
          <view class="setting-row">
            <text class="setting-label">显示色号</text>
            <switch :checked="showCode" @change="toggleCode" color="#ec4899" />
          </view>
          <view class="setting-row">
            <text class="setting-label">颜色筛选</text>
            <switch :checked="filterMode" @change="toggleFilter" color="#ec4899" />
          </view>
          <view class="setting-row">
            <text class="setting-label">颜色修改</text>
            <switch :checked="editMode" @change="toggleEdit" color="#ec4899" />
          </view>
          <view class="setting-row">
            <text class="setting-label">裁剪模式</text>
            <switch :checked="cropMode" @change="toggleCrop" color="#ec4899" />
          </view>
          <view class="setting-row full-width" v-if="cropMode">
            <text class="setting-label">选择区域</text>
            <button class="clear-filter-btn" @click="applyCrop" size="mini">
              <text>应用裁剪</text>
            </button>
            <button class="clear-filter-btn" @click="cancelCrop" size="mini">
              <text>取消</text>
            </button>
          </view>
          <view class="crop-tip" v-if="cropMode">
            <text class="tip-icon">💡</text>
            <text class="tip-text">点击两个对角点来框选裁剪区域</text>
          </view>
          <view class="setting-row full-width" v-if="editMode && selectedPixels.length > 0">
            <text class="setting-label">已选像素</text>
            <view class="selected-colors-badge">
              <text>{{ selectedPixels.length }} 个</text>
            </view>
            <button class="clear-filter-btn" @click="clearPixelSelection" size="mini">
              <text>清除</text>
            </button>
          </view>
          <view class="setting-row full-width" v-if="filterMode && selectedColors.length > 0">
            <text class="setting-label">已选颜色</text>
            <view class="selected-colors-badge">
              <text>{{ selectedColors.length }} 种</text>
            </view>
            <button class="clear-filter-btn" @click="clearFilter" size="mini">
              <text>清除</text>
            </button>
          </view>
        </view>
      </view>

      <scroll-view class="canvas-scroll-wrapper" scroll-x scroll-y enable-flex>
        <view class="canvas-container">
          <view
            class="pixel-grid-wrapper"
            :class="{ 'show-grid': showGrid, 'show-code': showCode }"
          >
            <view
              class="pixel-grid-container"
              v-if="showGrid"
              :style="{
                '--grid-cols': boardSize,
                '--grid-rows': gridRows
              }"
            >
              <view 
                class="row-numbers"
                :style="{
                  height: showCode ? gridHeight * 2 + 'px' : gridHeight + 'px'
                }"
              >
                <text
                  v-for="row in gridRows"
                  :key="'row-' + row"
                  class="row-number"
                >{{ row }}</text>
              </view>
              <view class="grid-with-columns">
                <view 
                  class="column-numbers"
                  :style="{
                    width: showCode ? gridWidth * 2 + 'px' : gridWidth + 'px'
                  }"
                >
                  <text
                    v-for="col in boardSize"
                    :key="'col-' + col"
                    class="column-number"
                  >{{ col }}</text>
                </view>
                <view
                  class="pixel-grid"
                  :class="{ 'with-code': showCode }"
                  :style="{
                    width: showCode ? gridWidth * 2 + 'px' : gridWidth + 'px',
                    height: showCode ? gridHeight * 2 + 'px' : gridHeight + 'px',
                    gridTemplateColumns: 'repeat(' + boardSize + ', 1fr)',
                    gridTemplateRows: 'repeat(' + gridRows + ', 1fr)'
                  }"
                >
                  <view
                    v-for="(color, index) in pixelData"
                    :key="index"
                    class="pixel-cell"
                    :class="{ 
                      'with-code': showCode, 
                      'filtered-out': filterMode && !isColorSelected(color),
                      'pixel-selected': editMode && selectedPixels.includes(index),
                      'crop-selected': cropMode && isInCropArea(index)
                    }"
                    :style="{ backgroundColor: filterMode && !isColorSelected(color) ? '#f0f0f0' : color }"
                    @click="handlePixelClick(index)"
                  >
                    <text v-if="showCode && (!filterMode || isColorSelected(color))" class="pixel-code">{{ getPixelCode(color) }}</text>
                  </view>
                </view>
              </view>
            </view>
            <view
              v-else
              class="pixel-grid"
              :class="{ 'with-code': showCode }"
              :style="{
                width: showCode ? gridWidth * 2 + 'px' : gridWidth + 'px',
                height: showCode ? gridHeight * 2 + 'px' : gridHeight + 'px',
                gridTemplateColumns: 'repeat(' + boardSize + ', 1fr)',
                gridTemplateRows: 'repeat(' + gridRows + ', 1fr)'
              }"
            >
              <view
                v-for="(color, index) in pixelData"
                :key="index"
                class="pixel-cell"
                :class="{ 
                  'with-code': showCode, 
                  'filtered-out': filterMode && !isColorSelected(color),
                  'pixel-selected': editMode && selectedPixels.includes(index),
                  'crop-selected': cropMode && isInCropArea(index)
                }"
                :style="{ backgroundColor: filterMode && !isColorSelected(color) ? '#f0f0f0' : color }"
                @click="handlePixelClick(index)"
              >
                <text v-if="showCode && (!filterMode || isColorSelected(color))" class="pixel-code">{{ getPixelCode(color) }}</text>
              </view>
            </view>
          </view>
        </view>
      </scroll-view>

      <view class="size-info" v-if="pixelData.length > 0">
        <view class="size-info-item">
          <text class="size-info-label">实际尺寸</text>
          <text class="size-info-value">{{ boardSize }} × {{ gridRows }} 像素</text>
        </view>
        <view class="size-info-divider"></view>
        <view class="size-info-item">
          <text class="size-info-label">总豆数</text>
          <text class="size-info-value">{{ pixelData.length }} 颗</text>
        </view>
      </view>

      <view class="info-section">
        <view class="info-card">
          <view class="info-header">
            <text class="info-title">📊 材料清单</text>
            <view class="info-badge">
              <text>{{ pixelData.length }} 颗豆</text>
            </view>
          </view>
          <scroll-view class="material-list" scroll-y="true">
            <view class="material-grid">
              <view
                v-for="(item, index) in materialList"
                :key="index"
                class="material-item"
                :class="{ 'selected': filterMode && isColorSelected(item.color) }"
                @click="filterMode && toggleColorSelect(item.color)"
              >
                <view class="color-swatch" :style="{ backgroundColor: item.color }">
                  <view v-if="filterMode && isColorSelected(item.color)" class="selected-mark">✓</view>
                </view>
                <view class="color-info">
                  <text class="color-code">{{ item.code }}</text>
                  <text class="color-name">{{ item.name }}</text>
                </view>
                <view class="color-count">
                  <text>{{ item.count }}</text>
                </view>
              </view>
            </view>
          </scroll-view>
        </view>
      </view>

      <view style="height: 180rpx;"></view>
    </scroll-view>

    <view class="preview-footer">
      <button type="default" @click="regenerate" class="btn-regenerate">
        <text>📷 重新选择</text>
      </button>
      <button 
        type="primary" 
        @click="openColorDrawer" 
        class="btn-color-picker"
        :disabled="!editMode || selectedPixels.length === 0"
        :class="{ disabled: !editMode || selectedPixels.length === 0 }"
      >
        <text v-if="selectedPixels.length > 0">🎨 选择颜色 ({{ selectedPixels.length }})</text>
        <text v-else>🎨 选择颜色</text>
      </button>
      <button type="primary" @click="exportImage" class="btn-export">
        <text>💾 导出图片</text>
      </button>
    </view>

    <view class="color-drawer-mask" v-if="drawerVisible" @click="closeColorDrawer"></view>
    <view class="color-drawer" :class="{ open: drawerVisible }">
      <view class="color-drawer-header">
        <text class="color-drawer-title">🎨 选择新颜色</text>
        <view class="color-drawer-close" @click="closeColorDrawer">
          <text>×</text>
        </view>
      </view>
      <scroll-view class="color-drawer-content" scroll-y>
        <view class="color-drawer-grid">
          <view
            v-for="(color, index) in mardColors"
            :key="index"
            class="color-drawer-item"
            @click="applyColorFromDrawer(color)"
          >
            <view 
              class="color-drawer-swatch" 
              :style="{ backgroundColor: `rgb(${color.r},${color.g},${color.b})` }"
            ></view>
            <view class="color-drawer-info">
              <text class="color-drawer-code">{{ color.code }}</text>
              <text class="color-drawer-name">{{ color.name }}</text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      originalImage: null,
      boardSize: 30,
      showGrid: true,
      showCode: false,
      filterMode: false,
      editMode: false,
      cropMode: false,
      drawerVisible: false,
      selectedColors: [],
      selectedPixels: [],
      cropStart: null,
      cropEnd: null,
      gridWidth: 300,
      gridHeight: 300,
      gridRows: 30,
      pixelSize: 12,
      pixelData: [],
      boardSizes: [
        { value: 10, label: '10×10' },
        { value: 20, label: '20×20' },
        { value: 30, label: '30×30' },
        { value: 40, label: '40×40' },
        { value: 50, label: '50×50' }
      ],
      mardColors: [
        { r: 250, g: 244, b: 200, name: '浅米黄', code: 'A1' },
        { r: 255, g: 255, b: 213, name: '浅奶黄', code: 'A2' },
        { r: 254, g: 255, b: 139, name: '浅亮黄', code: 'A3' },
        { r: 251, g: 237, b: 86, name: '亮黄', code: 'A4' },
        { r: 244, g: 215, b: 56, name: '黄', code: 'A5' },
        { r: 254, g: 172, b: 76, name: '浅橙黄', code: 'A6' },
        { r: 254, g: 139, b: 76, name: '浅橙', code: 'A7' },
        { r: 255, g: 218, b: 69, name: '中黄', code: 'A8' },
        { r: 255, g: 153, b: 91, name: '中橙', code: 'A9' },
        { r: 247, g: 124, b: 49, name: '深橙', code: 'A10' },
        { r: 255, g: 221, b: 153, name: '浅杏黄', code: 'A11' },
        { r: 254, g: 159, b: 114, name: '杏橙', code: 'A12' },
        { r: 255, g: 195, b: 101, name: '金橙', code: 'A13' },
        { r: 253, g: 84, b: 61, name: '亮橙红', code: 'A14' },
        { r: 255, g: 243, b: 101, name: '嫩黄', code: 'A15' },
        { r: 255, g: 255, b: 159, name: '浅鹅黄', code: 'A16' },
        { r: 255, g: 227, b: 110, name: '鹅黄', code: 'A17' },
        { r: 254, g: 190, b: 125, name: '浅金橙', code: 'A18' },
        { r: 253, g: 124, b: 114, name: '浅橙红', code: 'A19' },
        { r: 255, g: 213, b: 104, name: '金黄', code: 'A20' },
        { r: 255, g: 227, b: 149, name: '浅金黄', code: 'A21' },
        { r: 244, g: 245, b: 125, name: '亮黄绿', code: 'A22' },
        { r: 230, g: 201, b: 183, name: '浅棕黄', code: 'A23' },
        { r: 247, g: 248, b: 162, name: '浅柠黄', code: 'A24' },
        { r: 255, g: 214, b: 125, name: '金杏黄', code: 'A25' },
        { r: 255, g: 200, b: 48, name: '亮金', code: 'A26' },
        { r: 230, g: 238, b: 49, name: '亮草绿', code: 'B1' },
        { r: 99, g: 243, b: 71, name: '鲜绿', code: 'B2' },
        { r: 158, g: 247, b: 128, name: '浅亮绿', code: 'B3' },
        { r: 93, g: 224, b: 53, name: '草绿', code: 'B4' },
        { r: 53, g: 227, b: 82, name: '翠绿', code: 'B5' },
        { r: 101, g: 226, b: 166, name: '浅青翠绿', code: 'B6' },
        { r: 61, g: 175, b: 128, name: '青翠绿', code: 'B7' },
        { r: 28, g: 156, b: 79, name: '森林绿', code: 'B8' },
        { r: 39, g: 82, b: 58, name: '深林绿', code: 'B9' },
        { r: 149, g: 211, b: 194, name: '浅青灰', code: 'B10' },
        { r: 93, g: 114, b: 42, name: '暗绿', code: 'B11' },
        { r: 22, g: 111, b: 65, name: '墨绿', code: 'B12' },
        { r: 202, g: 235, b: 123, name: '浅黄绿', code: 'B13' },
        { r: 173, g: 233, b: 70, name: '鲜黄绿', code: 'B14' },
        { r: 46, g: 81, b: 50, name: '深暗绿', code: 'B15' },
        { r: 197, g: 237, b: 156, name: '浅绿', code: 'B16' },
        { r: 155, g: 177, b: 58, name: '橄榄绿', code: 'B17' },
        { r: 230, g: 238, b: 73, name: '亮黄绿', code: 'B18' },
        { r: 36, g: 184, b: 140, name: '青翠绿', code: 'B19' },
        { r: 194, g: 240, b: 204, name: '浅薄荷绿', code: 'B20' },
        { r: 21, g: 106, b: 107, name: '青灰绿', code: 'B21' },
        { r: 11, g: 60, b: 67, name: '深青灰', code: 'B22' },
        { r: 48, g: 58, b: 33, name: '暗深绿', code: 'B23' },
        { r: 238, g: 252, b: 165, name: '浅柠绿', code: 'B24' },
        { r: 78, g: 132, b: 109, name: '灰绿', code: 'B25' },
        { r: 141, g: 122, b: 53, name: '橄榄黄', code: 'B26' },
        { r: 204, g: 225, b: 175, name: '淡绿', code: 'B27' },
        { r: 158, g: 229, b: 185, name: '薄荷绿', code: 'B28' },
        { r: 197, g: 226, b: 84, name: '黄绿', code: 'B29' },
        { r: 226, g: 252, b: 177, name: '浅淡绿', code: 'B30' },
        { r: 176, g: 231, b: 146, name: '淡亮绿', code: 'B31' },
        { r: 156, g: 171, b: 90, name: '暗黄绿', code: 'B32' },
        { r: 232, g: 255, b: 231, name: '极浅青', code: 'C1' },
        { r: 169, g: 249, b: 252, name: '浅青', code: 'C2' },
        { r: 160, g: 226, b: 251, name: '浅天蓝', code: 'C3' },
        { r: 65, g: 204, b: 255, name: '亮蓝', code: 'C4' },
        { r: 1, g: 172, b: 235, name: '青蓝', code: 'C5' },
        { r: 80, g: 170, b: 240, name: '天蓝', code: 'C6' },
        { r: 54, g: 119, b: 210, name: '中蓝', code: 'C7' },
        { r: 15, g: 84, b: 192, name: '深蓝', code: 'C8' },
        { r: 50, g: 75, b: 202, name: '靛蓝', code: 'C9' },
        { r: 62, g: 188, b: 226, name: '亮青蓝', code: 'C10' },
        { r: 40, g: 221, b: 222, name: '亮青', code: 'C11' },
        { r: 28, g: 51, b: 77, name: '深青灰', code: 'C12' },
        { r: 205, g: 232, b: 255, name: '浅天青', code: 'C13' },
        { r: 213, g: 253, b: 255, name: '浅亮青', code: 'C14' },
        { r: 34, g: 196, b: 198, name: '青翠绿', code: 'C15' },
        { r: 21, g: 87, b: 168, name: '深天蓝', code: 'C16' },
        { r: 4, g: 209, b: 246, name: '亮天青', code: 'C17' },
        { r: 29, g: 51, b: 68, name: '深灰蓝', code: 'C18' },
        { r: 24, g: 135, b: 162, name: '灰蓝', code: 'C19' },
        { r: 23, g: 109, b: 175, name: '中灰蓝', code: 'C20' },
        { r: 190, g: 221, b: 255, name: '淡天蓝', code: 'C21' },
        { r: 103, g: 180, b: 190, name: '灰青', code: 'C22' },
        { r: 200, g: 226, b: 255, name: '浅淡蓝', code: 'C23' },
        { r: 124, g: 196, b: 255, name: '淡蓝', code: 'C24' },
        { r: 169, g: 229, b: 229, name: '浅灰青', code: 'C25' },
        { r: 60, g: 174, b: 216, name: '青蓝', code: 'C26' },
        { r: 211, g: 223, b: 250, name: '浅蓝灰', code: 'C27' },
        { r: 187, g: 207, b: 237, name: '淡蓝灰', code: 'C28' },
        { r: 52, g: 72, b: 142, name: '深靛蓝', code: 'C29' },
        { r: 174, g: 180, b: 242, name: '浅紫蓝', code: 'D1' },
        { r: 133, g: 142, b: 221, name: '紫蓝', code: 'D2' },
        { r: 47, g: 84, b: 175, name: '深紫蓝', code: 'D3' },
        { r: 24, g: 42, b: 132, name: '靛蓝紫', code: 'D4' },
        { r: 184, g: 67, b: 197, name: '亮紫', code: 'D5' },
        { r: 172, g: 123, b: 222, name: '浅紫', code: 'D6' },
        { r: 136, g: 84, b: 179, name: '紫', code: 'D7' },
        { r: 226, g: 211, b: 255, name: '淡紫', code: 'D8' },
        { r: 213, g: 185, b: 248, name: '浅淡紫', code: 'D9' },
        { r: 54, g: 24, b: 81, name: '深紫', code: 'D10' },
        { r: 185, g: 186, b: 225, name: '浅紫灰', code: 'D11' },
        { r: 222, g: 154, b: 212, name: '粉紫', code: 'D12' },
        { r: 185, g: 0, b: 149, name: '深玫紫', code: 'D13' },
        { r: 139, g: 39, b: 155, name: '玫紫', code: 'D14' },
        { r: 47, g: 31, b: 144, name: '靛紫', code: 'D15' },
        { r: 227, g: 225, b: 238, name: '极浅紫', code: 'D16' },
        { r: 196, g: 212, b: 246, name: '浅蓝紫', code: 'D17' },
        { r: 164, g: 94, b: 199, name: '亮紫', code: 'D18' },
        { r: 216, g: 195, b: 215, name: '淡灰紫', code: 'D19' },
        { r: 156, g: 50, b: 178, name: '深亮紫', code: 'D20' },
        { r: 154, g: 0, b: 155, name: '深玫紫', code: 'D21' },
        { r: 51, g: 58, b: 149, name: '深靛蓝', code: 'D22' },
        { r: 235, g: 218, b: 252, name: '淡紫', code: 'D23' },
        { r: 119, g: 134, b: 229, name: '灰蓝紫', code: 'D24' },
        { r: 73, g: 79, b: 199, name: '靛紫蓝', code: 'D25' },
        { r: 223, g: 194, b: 248, name: '浅粉紫', code: 'D26' },
        { r: 253, g: 211, b: 204, name: '浅粉红', code: 'E1' },
        { r: 254, g: 192, b: 223, name: '浅粉', code: 'E2' },
        { r: 255, g: 183, b: 231, name: '亮粉', code: 'E3' },
        { r: 232, g: 100, b: 158, name: '玫瑰粉', code: 'E4' },
        { r: 245, g: 81, b: 162, name: '亮玫瑰', code: 'E5' },
        { r: 241, g: 61, b: 116, name: '玫红', code: 'E6' },
        { r: 198, g: 52, b: 120, name: '深玫红', code: 'E7' },
        { r: 255, g: 219, b: 233, name: '淡粉红', code: 'E8' },
        { r: 233, g: 112, b: 204, name: '亮玫粉', code: 'E9' },
        { r: 211, g: 55, b: 147, name: '亮玫红', code: 'E10' },
        { r: 252, g: 221, b: 210, name: '淡粉桃', code: 'E11' },
        { r: 247, g: 143, b: 195, name: '玫瑰粉', code: 'E12' },
        { r: 181, g: 0, b: 109, name: '深玫红', code: 'E13' },
        { r: 255, g: 209, b: 186, name: '浅桃粉', code: 'E14' },
        { r: 248, g: 199, b: 201, name: '桃粉', code: 'E15' },
        { r: 255, g: 243, b: 235, name: '极浅粉', code: 'E16' },
        { r: 255, g: 226, b: 234, name: '淡粉', code: 'E17' },
        { r: 255, g: 199, b: 219, name: '浅玫瑰粉', code: 'E18' },
        { r: 254, g: 186, b: 213, name: '玫瑰粉', code: 'E19' },
        { r: 216, g: 199, b: 209, name: '灰粉', code: 'E20' },
        { r: 189, g: 157, b: 161, name: '暗粉', code: 'E21' },
        { r: 183, g: 133, b: 161, name: '暗玫瑰粉', code: 'E22' },
        { r: 147, g: 122, b: 141, name: '深灰粉', code: 'E23' },
        { r: 225, g: 188, b: 232, name: '淡紫粉', code: 'E24' },
        { r: 253, g: 149, b: 123, name: '浅橘粉', code: 'F1' },
        { r: 252, g: 61, b: 70, name: '亮红', code: 'F2' },
        { r: 247, g: 73, b: 65, name: '红橙', code: 'F3' },
        { r: 252, g: 40, b: 60, name: '亮红', code: 'F4' },
        { r: 231, g: 0, b: 47, name: '深红', code: 'F5' },
        { r: 148, g: 54, b: 48, name: '暗红棕', code: 'F6' },
        { r: 151, g: 25, b: 55, name: '深酒红', code: 'F7' },
        { r: 188, g: 0, b: 40, name: '酒红', code: 'F8' },
        { r: 226, g: 103, b: 122, name: '浅酒红', code: 'F9' },
        { r: 138, g: 69, b: 38, name: '暗红', code: 'F10' },
        { r: 90, g: 33, b: 33, name: '深暗红', code: 'F11' },
        { r: 253, g: 78, b: 106, name: '亮粉红', code: 'F12' },
        { r: 243, g: 87, b: 68, name: '亮红橙', code: 'F13' },
        { r: 255, g: 169, b: 173, name: '浅粉橙', code: 'F14' },
        { r: 211, g: 0, b: 34, name: '大红', code: 'F15' },
        { r: 254, g: 194, b: 166, name: '浅桃橙', code: 'F16' },
        { r: 230, g: 156, b: 121, name: '桃橙', code: 'F17' },
        { r: 211, g: 124, b: 70, name: '暗桃橙', code: 'F18' },
        { r: 193, g: 68, b: 74, name: '暗酒红', code: 'F19' },
        { r: 205, g: 147, b: 145, name: '灰红', code: 'F20' },
        { r: 247, g: 180, b: 198, name: '浅粉红', code: 'F21' },
        { r: 253, g: 192, b: 208, name: '淡粉红', code: 'F22' },
        { r: 246, g: 126, b: 102, name: '亮橙红', code: 'F23' },
        { r: 230, g: 152, b: 170, name: '灰粉红', code: 'F24' },
        { r: 229, g: 75, b: 79, name: '亮酒红', code: 'F25' },
        { r: 255, g: 226, b: 206, name: '浅肤色', code: 'G1' },
        { r: 255, g: 196, b: 170, name: '浅桃色', code: 'G2' },
        { r: 244, g: 195, b: 165, name: '桃色', code: 'G3' },
        { r: 225, g: 179, b: 131, name: '浅棕黄', code: 'G4' },
        { r: 237, g: 176, b: 69, name: '金棕', code: 'G5' },
        { r: 233, g: 156, b: 23, name: '深金棕', code: 'G6' },
        { r: 157, g: 91, b: 62, name: '暗棕', code: 'G7' },
        { r: 117, g: 56, b: 50, name: '深暗棕', code: 'G8' },
        { r: 230, g: 180, b: 131, name: '浅棕', code: 'G9' },
        { r: 217, g: 140, b: 57, name: '暗金棕', code: 'G10' },
        { r: 224, g: 197, b: 147, name: '淡棕黄', code: 'G11' },
        { r: 255, g: 200, b: 144, name: '浅金棕', code: 'G12' },
        { r: 183, g: 113, b: 74, name: '暗桃棕', code: 'G13' },
        { r: 141, g: 97, b: 76, name: '灰棕', code: 'G14' },
        { r: 252, g: 249, b: 224, name: '极浅肤', code: 'G15' },
        { r: 242, g: 217, b: 186, name: '淡肤', code: 'G16' },
        { r: 120, g: 82, b: 75, name: '深灰棕', code: 'G17' },
        { r: 255, g: 228, b: 204, name: '浅肤色', code: 'G18' },
        { r: 224, g: 121, b: 53, name: '亮棕', code: 'G19' },
        { r: 169, g: 64, b: 35, name: '深暗棕', code: 'G20' },
        { r: 184, g: 133, b: 88, name: '暗棕', code: 'G21' },
        { r: 253, g: 251, b: 255, name: '极浅白', code: 'H1' },
        { r: 254, g: 255, b: 255, name: '白', code: 'H2' },
        { r: 182, g: 177, b: 186, name: '浅灰紫', code: 'H3' },
        { r: 137, g: 133, b: 140, name: '灰', code: 'H4' },
        { r: 72, g: 70, b: 78, name: '深灰', code: 'H5' },
        { r: 47, g: 43, b: 47, name: '暗灰', code: 'H6' },
        { r: 0, g: 0, b: 0, name: '黑', code: 'H7' },
        { r: 231, g: 214, b: 219, name: '浅粉灰', code: 'H8' },
        { r: 237, g: 237, b: 237, name: '浅灰', code: 'H9' },
        { r: 238, g: 233, b: 234, name: '淡灰', code: 'H10' },
        { r: 206, g: 205, b: 213, name: '中灰', code: 'H11' },
        { r: 255, g: 245, b: 237, name: '浅米白', code: 'H12' },
        { r: 245, g: 236, b: 210, name: '米白', code: 'H13' },
        { r: 207, g: 215, b: 211, name: '浅青灰', code: 'H14' },
        { r: 152, g: 166, b: 168, name: '青灰', code: 'H15' },
        { r: 29, g: 20, b: 20, name: '深暗灰', code: 'H16' },
        { r: 241, g: 237, b: 237, name: '淡浅灰', code: 'H17' },
        { r: 255, g: 253, b: 240, name: '极浅米', code: 'H18' },
        { r: 246, g: 239, b: 226, name: '浅米灰', code: 'H19' },
        { r: 148, g: 159, b: 163, name: '暗青灰', code: 'H20' },
        { r: 255, g: 251, b: 225, name: '浅柠白', code: 'H21' },
        { r: 202, g: 202, b: 212, name: '中灰', code: 'H22' },
        { r: 154, g: 157, b: 148, name: '暗黄绿', code: 'H23' },
        { r: 188, g: 198, b: 184, name: '浅绿灰', code: 'M1' },
        { r: 138, g: 163, b: 134, name: '绿灰', code: 'M2' },
        { r: 105, g: 125, b: 128, name: '深绿灰', code: 'M3' },
        { r: 227, g: 210, b: 188, name: '浅棕灰', code: 'M4' },
        { r: 208, g: 204, b: 170, name: '棕黄灰', code: 'M5' },
        { r: 176, g: 167, b: 130, name: '暗棕灰', code: 'M6' },
        { r: 180, g: 164, b: 151, name: '棕灰', code: 'M7' },
        { r: 179, g: 130, b: 129, name: '暗红灰', code: 'M8' },
        { r: 165, g: 135, b: 103, name: '暗棕灰', code: 'M9' },
        { r: 197, g: 178, b: 188, name: '淡粉紫灰', code: 'M10' },
        { r: 159, g: 117, b: 148, name: '暗粉紫灰', code: 'M11' },
        { r: 100, g: 71, b: 73, name: '深暗灰', code: 'M12' },
        { r: 209, g: 144, b: 102, name: '亮棕灰', code: 'M13' },
        { r: 199, g: 115, b: 98, name: '橙棕灰', code: 'M14' },
        { r: 117, g: 125, b: 120, name: '深灰绿', code: 'M15' }
      ]
    }
  },
  computed: {
    materialList() {
      const countMap = {}
      this.pixelData.forEach(color => {
        if (!countMap[color]) {
          countMap[color] = 0
        }
        countMap[color]++
      })

      const list = []
      for (const color in countMap) {
        const matchedColor = this.mardColors.find(c => `rgb(${c.r},${c.g},${c.b})` === color)
        list.push({
          color: color,
          name: matchedColor ? matchedColor.name : '未知',
          code: matchedColor ? matchedColor.code : '-',
          count: countMap[color]
        })
      }

      return list.sort((a, b) => b.count - a.count)
    }
  },
  onLoad(options) {
    if (options.imageData) {
      try {
        this.originalImage = JSON.parse(decodeURIComponent(options.imageData))
        this.$nextTick(() => {
          this.generatePixelArt()
        })
      } catch (e) {
        console.error('Failed to parse image data:', e)
        uni.showToast({ title: '数据加载失败', icon: 'none' })
      }
    }
  },
  methods: {
    selectSize(size) {
      this.boardSize = size
      this.generatePixelArt()
    },
    toggleGrid(e) {
      this.showGrid = e.detail.value
    },
    toggleCode(e) {
      this.showCode = e.detail.value
    },
    toggleFilter(e) {
      this.filterMode = e.detail.value
      if (!this.filterMode) {
        this.selectedColors = []
      }
    },
    toggleColorSelect(color) {
      const index = this.selectedColors.indexOf(color)
      if (index > -1) {
        this.selectedColors.splice(index, 1)
      } else {
        this.selectedColors.push(color)
      }
    },
    isColorSelected(color) {
      return this.selectedColors.includes(color)
    },
    clearFilter() {
      this.selectedColors = []
    },
    toggleEdit(e) {
      this.editMode = e.detail.value
      if (!this.editMode) {
        this.selectedPixels = []
      }
    },
    togglePixelSelect(index) {
      const idx = this.selectedPixels.indexOf(index)
      if (idx > -1) {
        this.selectedPixels.splice(idx, 1)
      } else {
        this.selectedPixels.push(index)
      }
    },
    clearPixelSelection() {
      this.selectedPixels = []
    },
    toggleCrop(e) {
      this.cropMode = e.detail.value
      if (!this.cropMode) {
        this.cropStart = null
        this.cropEnd = null
      }
    },
    handlePixelClick(index) {
      if (this.editMode) {
        this.togglePixelSelect(index)
      } else if (this.cropMode) {
        this.selectCropArea(index)
      }
    },
    selectCropArea(index) {
      const row = Math.floor(index / this.boardSize)
      const col = index % this.boardSize
      
      if (!this.cropStart) {
        this.cropStart = { row, col }
        this.cropEnd = { row, col }
      } else {
        this.cropEnd = { row, col }
      }
    },
    isInCropArea(index) {
      if (!this.cropStart || !this.cropEnd) return false
      
      const row = Math.floor(index / this.boardSize)
      const col = index % this.boardSize
      
      const minRow = Math.min(this.cropStart.row, this.cropEnd.row)
      const maxRow = Math.max(this.cropStart.row, this.cropEnd.row)
      const minCol = Math.min(this.cropStart.col, this.cropEnd.col)
      const maxCol = Math.max(this.cropStart.col, this.cropEnd.col)
      
      return row >= minRow && row <= maxRow && col >= minCol && col <= maxCol
    },
    applyCrop() {
      if (!this.cropStart || !this.cropEnd) {
        uni.showToast({ title: '请先选择裁剪区域', icon: 'none' })
        return
      }
      
      const minRow = Math.min(this.cropStart.row, this.cropEnd.row)
      const maxRow = Math.max(this.cropStart.row, this.cropEnd.row)
      const minCol = Math.min(this.cropStart.col, this.cropEnd.col)
      const maxCol = Math.max(this.cropStart.col, this.cropEnd.col)
      
      const newWidth = maxCol - minCol + 1
      const newHeight = maxRow - minRow + 1
      
      const newPixelData = []
      for (let row = minRow; row <= maxRow; row++) {
        for (let col = minCol; col <= maxCol; col++) {
          const index = row * this.boardSize + col
          newPixelData.push(this.pixelData[index])
        }
      }
      
      this.pixelData = newPixelData
      this.boardSize = newWidth
      this.gridRows = newHeight
      this.gridWidth = newWidth * this.pixelSize
      this.gridHeight = newHeight * this.pixelSize
      
      this.cropMode = false
      this.cropStart = null
      this.cropEnd = null
      
      uni.showToast({ title: '裁剪成功', icon: 'success' })
    },
    cancelCrop() {
      this.cropMode = false
      this.cropStart = null
      this.cropEnd = null
    },
    openColorDrawer() {
      if (!this.editMode) {
        uni.showToast({ title: '请先开启「颜色修改」模式', icon: 'none' })
        return
      }
      if (this.selectedPixels.length === 0) {
        uni.showToast({ title: '请先点击选择像素', icon: 'none' })
        return
      }
      this.drawerVisible = true
    },
    closeColorDrawer() {
      this.drawerVisible = false
    },
    applyColorFromDrawer(color) {
      const newColor = `rgb(${color.r},${color.g},${color.b})`
      // 使用Vue的响应式方法来确保更新
      const updatedData = [...this.pixelData]
      this.selectedPixels.forEach(index => {
        updatedData[index] = newColor
      })
      this.pixelData = updatedData
      
      uni.showToast({ title: `已替换${this.selectedPixels.length}个像素`, icon: 'success' })
      this.selectedPixels = []
      this.drawerVisible = false
    },
    applyColorToSelected(color) {
      const newColor = `rgb(${color.r},${color.g},${color.b})`
      this.selectedPixels.forEach(index => {
        this.pixelData[index] = newColor
      })
      uni.showToast({ title: `已替换${this.selectedPixels.length}个像素`, icon: 'success' })
      this.selectedPixels = []
    },
    getPixelCode(color) {
      const matchedColor = this.mardColors.find(c => `rgb(${c.r},${c.g},${c.b})` === color)
      return matchedColor ? matchedColor.code : '?'
    },
    generatePixelArt() {
      if (!this.originalImage) return

      uni.showLoading({ title: '生成中...' })

      const self = this
      const img = new Image()
      img.crossOrigin = 'anonymous'
      img.onload = function() {
        const cols = self.boardSize
        const rows = Math.floor(cols * img.height / img.width)
        self.gridRows = rows
        self.gridWidth = cols * self.pixelSize
        self.gridHeight = rows * self.pixelSize

        const tempCanvas = document.createElement('canvas')
        const tempCtx = tempCanvas.getContext('2d')
        tempCanvas.width = cols
        tempCanvas.height = rows
        tempCtx.imageSmoothingEnabled = false
        tempCtx.drawImage(img, 0, 0, cols, rows)

        const imageData = tempCtx.getImageData(0, 0, cols, rows)
        const data = imageData.data

        self.pixelData = []
        for (let y = 0; y < rows; y++) {
          for (let x = 0; x < cols; x++) {
            const idx = (y * cols + x) * 4
            const r = data[idx]
            const g = data[idx + 1]
            const b = data[idx + 2]
            const color = self.findClosestColor({ r, g, b })
            self.pixelData.push(`rgb(${color.r},${color.g},${color.b})`)
          }
        }

        uni.hideLoading()
      }
      img.onerror = function() {
        uni.hideLoading()
        uni.showToast({ title: '图片加载失败', icon: 'none' })
      }
      img.src = this.originalImage.tempFilePath
    },
    findClosestColor(target) {
      let closest = this.mardColors[0]
      let minDistance = Infinity

      for (const color of this.mardColors) {
        const distance = this.colorDistance(target, color)
        if (distance < minDistance) {
          minDistance = distance
          closest = color
        }
      }
      return closest
    },
    colorDistance(c1, c2) {
      const r1 = c1.r / 255
      const g1 = c1.g / 255
      const b1 = c1.b / 255
      const r2 = c2.r / 255
      const g2 = c2.g / 255
      const b2 = c2.b / 255

      const meanR = (r1 + r2) / 2

      const rDiff = r1 - r2
      const gDiff = g1 - g2
      const bDiff = b1 - b2

      const distance = (2 + meanR) * rDiff * rDiff + 4 * gDiff * gDiff + (2 + (1 - meanR)) * bDiff * bDiff
      return distance
    },
    regenerate() {
      uni.navigateBack()
    },
    exportImage() {
      try {
        const cols = this.boardSize
        const rows = this.gridRows
        const pixelSize = this.showCode ? this.pixelSize * 2 : this.pixelSize
        const numberWidth = this.showGrid ? 20 : 0
        const numberHeight = this.showGrid ? 20 : 0

        const exportCanvas = document.createElement('canvas')
        exportCanvas.width = cols * pixelSize + numberWidth
        exportCanvas.height = rows * pixelSize + numberHeight
        const ctx = exportCanvas.getContext('2d')

        ctx.fillStyle = '#ffffff'
        ctx.fillRect(0, 0, exportCanvas.width, exportCanvas.height)

        if (this.showGrid) {
          ctx.fillStyle = 'linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%)'
          ctx.fillStyle = '#ec4899'
          ctx.fillRect(0, numberHeight, numberWidth, rows * pixelSize)
          ctx.fillRect(numberWidth, 0, cols * pixelSize, numberHeight)

          ctx.fillStyle = '#ffffff'
          ctx.font = `bold ${Math.min(numberWidth, numberHeight) * 0.6}px Arial`
          ctx.textAlign = 'center'
          ctx.textBaseline = 'middle'

          for (let y = 0; y < rows; y++) {
            ctx.fillText((y + 1).toString(), numberWidth / 2, numberHeight + y * pixelSize + pixelSize / 2)
          }

          for (let x = 0; x < cols; x++) {
            ctx.fillText((x + 1).toString(), numberWidth + x * pixelSize + pixelSize / 2, numberHeight / 2)
          }
        }

        for (let y = 0; y < rows; y++) {
          for (let x = 0; x < cols; x++) {
            const color = this.pixelData[y * cols + x]
            const isSelected = !this.filterMode || this.isColorSelected(color)
            
            if (isSelected) {
              ctx.fillStyle = color
              ctx.fillRect(numberWidth + x * pixelSize, numberHeight + y * pixelSize, pixelSize, pixelSize)

              if (this.showCode) {
                const code = this.getPixelCode(color)
                ctx.fillStyle = this.getContrastColor(color)
                ctx.font = `bold ${pixelSize * 0.3}px Arial`
                ctx.textAlign = 'center'
                ctx.textBaseline = 'middle'
                ctx.fillText(code, numberWidth + x * pixelSize + pixelSize / 2, numberHeight + y * pixelSize + pixelSize / 2)
              }
            } else {
              ctx.fillStyle = '#f0f0f0'
              ctx.fillRect(numberWidth + x * pixelSize, numberHeight + y * pixelSize, pixelSize, pixelSize)
            }
          }
        }

        const dataUrl = exportCanvas.toDataURL('image/png')

        const link = document.createElement('a')
        link.download = `pixel-art-MARD-${this.boardSize}x${this.gridRows}${this.showCode ? '-code' : ''}${this.showGrid ? '-numbers' : ''}.png`
        link.href = dataUrl
        link.click()

        uni.showToast({ title: '保存成功', icon: 'success' })
      } catch (error) {
        console.error(error)
        uni.showToast({ title: '保存失败', icon: 'none' })
      }
    },
    getContrastColor(color) {
      const match = color.match(/rgb\((\d+),(\d+),(\d+)\)/)
      if (!match) return '#000000'
      const r = parseInt(match[1])
      const g = parseInt(match[2])
      const b = parseInt(match[3])
      const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255
      return luminance > 0.5 ? '#000000' : '#ffffff'
    }
  }
}
</script>

<style scoped>
.preview-page {
  height: 100vh;
  background: linear-gradient(135deg, #fff0f6 0%, #fce7f3 25%, #f3e8ff 50%, #e0e7ff 75%, #dbeafe 100%);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.preview-page::before {
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

.preview-content {
  flex: 1;
  overflow-y: auto;
  padding: 20rpx;
  padding-bottom: 0;
  position: relative;
  z-index: 1;
}

.preview-header {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.98) 0%, rgba(253, 242, 248, 0.98) 100%);
  border-radius: 32rpx;
  padding: calc(10rpx + env(safe-area-inset-top)) 24rpx 24rpx;
  margin-bottom: 20rpx;
  backdrop-filter: blur(20px);
  box-shadow: 0 8rpx 32rpx rgba(236, 72, 153, 0.12), 0 4rpx 16rpx rgba(168, 85, 247, 0.08);
  border: 2rpx solid rgba(255, 255, 255, 0.8);
}

.header-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24rpx;
}

.back-btn {
  width: 80rpx;
  height: 80rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 56rpx;
  color: #ec4899;
  font-weight: 300;
  background: linear-gradient(135deg, #fdf2f8 0%, #fce7f3 100%);
  border-radius: 20rpx;
  box-shadow: 0 4rpx 12rpx rgba(236, 72, 153, 0.15);
  transition: all 0.2s;
}

.back-btn:active {
  transform: scale(0.95);
  box-shadow: 0 2rpx 6rpx rgba(236, 72, 153, 0.2);
}

.title {
  font-size: 42rpx;
  font-weight: 800;
  color: #333;
  background: linear-gradient(135deg, #ec4899 0%, #f472b6 25%, #a78bfa 50%, #818cf8 75%, #60a5fa 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 4rpx 8rpx rgba(236, 72, 153, 0.2);
}

.settings {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16rpx;
}

.setting-row {
  display: flex;
  align-items: center;
  gap: 12rpx;
  padding: 16rpx;
  background: linear-gradient(135deg, #fafafa 0%, #f5f5f5 100%);
  border-radius: 20rpx;
  border: 1rpx solid rgba(236, 72, 153, 0.1);
}

.setting-row.full-width {
  grid-column: span 2;
}

.setting-label {
  font-size: 28rpx;
  color: #333;
  font-weight: 600;
  min-width: 140rpx;
}

.size-options {
  display: flex;
  gap: 12rpx;
  flex-wrap: wrap;
  flex: 1;
}

.size-option {
  padding: 12rpx 24rpx;
  background: linear-gradient(135deg, #ffffff 0%, #fafafa 100%);
  border-radius: 16rpx;
  font-size: 26rpx;
  color: #666;
  transition: all 0.25s;
  border: 2rpx solid #e5e7eb;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.04);
}

.size-option:active {
  transform: scale(0.96);
}

.size-option.active {
  background: linear-gradient(135deg, #ec4899 0%, #f472b6 50%, #a78bfa 100%);
  color: #fff;
  border-color: transparent;
  box-shadow: 0 4rpx 16rpx rgba(236, 72, 153, 0.35);
  transform: scale(1.02);
}

.canvas-scroll-wrapper {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.98) 0%, rgba(253, 242, 248, 0.98) 100%);
  border-radius: 32rpx;
  margin-bottom: 20rpx;
  backdrop-filter: blur(20px);
  box-shadow: 0 8rpx 32rpx rgba(236, 72, 153, 0.12), 0 4rpx 16rpx rgba(168, 85, 247, 0.08);
  max-height: 60vh;
  overflow-x: auto !important;
  overflow-y: auto !important;
  -webkit-overflow-scrolling: touch;
  border: 2rpx solid rgba(255, 255, 255, 0.8);
}

.canvas-container {
  padding: 32rpx;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: auto;
  min-width: 100%;
}

.pixel-grid-wrapper {
  border-radius: 24rpx;
  overflow: hidden;
  box-shadow: 0 12rpx 48rpx rgba(236, 72, 153, 0.18), 0 6rpx 24rpx rgba(168, 85, 247, 0.12);
}

.pixel-grid-container {
  display: flex;
  gap: 0;
}

.row-numbers {
  display: grid;
  grid-template-rows: repeat(var(--grid-rows), 1fr);
  background: linear-gradient(135deg, #f472b6 0%, #ec4899 50%, #f43f5e 100%);
  border-radius: 20rpx 0 0 20rpx;
  padding: 0 8rpx;
  align-items: center;
  margin-top: 40rpx;
  box-shadow: inset 0 0 20rpx rgba(255, 255, 255, 0.15);
}

.row-number {
  font-size: 11px;
  font-weight: 800;
  color: #fff;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
}

.grid-with-columns {
  display: flex;
  flex-direction: column;
}

.column-numbers {
  display: grid;
  grid-template-columns: repeat(var(--grid-cols), 1fr);
  background: linear-gradient(135deg, #f472b6 0%, #ec4899 50%, #f43f5e 100%);
  border-radius: 0 20rpx 0 0;
  padding: 8rpx 0;
  align-items: center;
  justify-items: center;
  box-shadow: inset 0 0 20rpx rgba(255, 255, 255, 0.15);
}

.column-number {
  font-size: 11px;
  font-weight: 800;
  color: #fff;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
}

.pixel-grid-wrapper.show-grid .pixel-grid {
  border: 2px solid rgba(236, 72, 153, 0.2);
}

.pixel-grid {
  display: grid;
  background: linear-gradient(135deg, #f3f4f6 0%, #e5e7eb 100%);
  gap: 1.5px;
}

.pixel-grid.with-code {
  gap: 0;
}

.pixel-cell {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s ease;
  position: relative;
  cursor: pointer;
  pointer-events: auto;
}

.pixel-cell:active {
  transform: scale(0.95);
}

.pixel-cell.with-code {
  border: 1px solid rgba(236, 72, 153, 0.1);
}

.pixel-cell.filtered-out {
  opacity: 0.6;
  background-color: #e0e0e0 !important;
}

.pixel-cell.pixel-selected {
  border: 4px solid #f472b6 !important;
  box-shadow: 0 0 0 3px rgba(244, 114, 182, 0.4), 0 2px 8px rgba(236, 72, 153, 0.3);
  z-index: 10;
}

.pixel-cell.crop-selected {
  border: 4px solid #34d399 !important;
  box-shadow: 0 0 0 3px rgba(52, 211, 153, 0.4), 0 2px 8px rgba(16, 185, 129, 0.3);
  z-index: 10;
}

.pixel-code {
  font-size: 10px;
  font-weight: bold;
  font-family: Arial, sans-serif;
  text-align: center;
  line-height: 1;
  color: inherit;
  text-shadow: 0 0 2px rgba(255, 255, 255, 0.8);
}

.size-info {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24rpx;
  padding: 24rpx;
  margin-bottom: 20rpx;
  backdrop-filter: blur(10px);
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 30rpx;
}

.size-info-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8rpx;
}

.size-info-label {
  font-size: 24rpx;
  color: #999;
}

.size-info-value {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  background: linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.size-info-divider {
  width: 2rpx;
  height: 60rpx;
  background: #e0e0e0;
}

.selected-colors-badge {
  background: linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%);
  color: #fff;
  padding: 6rpx 16rpx;
  border-radius: 16rpx;
  font-size: 22rpx;
  font-weight: 500;
}

.clear-filter-btn {
  background: #f5f5f5;
  color: #666;
  border: none;
  padding: 4rpx 16rpx;
  border-radius: 16rpx;
  font-size: 22rpx;
  margin-left: 10rpx;
}

.crop-tip {
  display: flex;
  align-items: center;
  gap: 12rpx;
  padding: 16rpx 20rpx;
  background: linear-gradient(135deg, rgba(16, 185, 129, 0.1) 0%, rgba(34, 211, 238, 0.1) 100%);
  border-radius: 20rpx;
  margin-top: 16rpx;
  border: 2rpx solid rgba(16, 185, 129, 0.2);
  grid-column: span 2;
}

.tip-icon {
  font-size: 36rpx;
}

.tip-text {
  font-size: 26rpx;
  color: #059669;
  font-weight: 500;
  line-height: 1.4;
}

.info-section {
  margin-bottom: 20rpx;
}

.info-card {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.98) 0%, rgba(253, 242, 248, 0.98) 100%);
  border-radius: 32rpx;
  padding: 32rpx;
  backdrop-filter: blur(20px);
  box-shadow: 0 8rpx 32rpx rgba(236, 72, 153, 0.12), 0 4rpx 16rpx rgba(168, 85, 247, 0.08);
  border: 2rpx solid rgba(255, 255, 255, 0.8);
}

.info-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}

.info-title {
  font-size: 30rpx;
  font-weight: bold;
  color: #333;
}

.info-badge {
  background: linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%);
  color: #fff;
  padding: 8rpx 16rpx;
  border-radius: 20rpx;
  font-size: 22rpx;
  font-weight: 500;
}

.material-list {
  max-height: 360rpx;
  overflow-y: auto;
}

.material-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16rpx;
}

.material-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8rpx;
  padding: 16rpx;
  background: #f8f9fa;
  border-radius: 12rpx;
  transition: all 0.2s;
}

.material-item.selected {
  background: rgba(236, 72, 153, 0.15);
  border: 2rpx solid #ec4899;
}

.color-swatch {
  width: 60rpx;
  height: 60rpx;
  border-radius: 10rpx;
  border: 2rpx solid #eee;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.06);
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.selected-mark {
  color: #fff;
  font-size: 24rpx;
  font-weight: bold;
  text-shadow: 0 1rpx 2rpx rgba(0, 0, 0, 0.3);
}

.color-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4rpx;
}

.color-name {
  font-size: 20rpx;
  color: #999;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 100%;
}

.color-code {
  font-size: 24rpx;
  color: #333;
  font-weight: bold;
  font-family: monospace;
}

.color-count {
  font-size: 22rpx;
  color: #ec4899;
  font-weight: bold;
  background: rgba(236, 72, 153, 0.1);
  padding: 4rpx 12rpx;
  border-radius: 16rpx;
}

.preview-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 28rpx 32rpx;
  padding-bottom: calc(28rpx + env(safe-area-inset-bottom));
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.98) 0%, rgba(253, 242, 248, 0.98) 100%);
  backdrop-filter: blur(20px);
  display: flex;
  gap: 16rpx;
  box-shadow: 0 -8rpx 32rpx rgba(236, 72, 153, 0.15), 0 -4rpx 16rpx rgba(168, 85, 247, 0.1);
  border-radius: 32rpx 32rpx 0 0;
  border-top: 2rpx solid rgba(255, 255, 255, 0.8);
  z-index: 999;
  pointer-events: auto;
}

.btn-regenerate,
.btn-export,
.btn-color-picker {
  flex: 1;
  height: 104rpx;
  line-height: 104rpx;
  border-radius: 52rpx;
  font-size: 32rpx;
  font-weight: 700;
  border: none;
  transition: all 0.25s ease;
  position: relative;
  overflow: hidden;
  pointer-events: auto !important;
  cursor: pointer;
  z-index: 1;
}

.btn-regenerate::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.8) 0%, rgba(255, 255, 255, 0.2) 100%);
  pointer-events: none;
  z-index: -1;
}

.btn-regenerate {
  background: linear-gradient(135deg, #f9fafb 0%, #f3f4f6 50%, #e5e7eb 100%);
  color: #4b5563;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.06), inset 0 1px 0 rgba(255, 255, 255, 0.8);
  border: 2rpx solid #e5e7eb;
}

.btn-regenerate:active {
  transform: scale(0.97);
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.08), inset 0 2px 4px rgba(0, 0, 0, 0.05);
}

.btn-export {
  background: linear-gradient(135deg, #ec4899 0%, #f472b6 30%, #a78bfa 70%, #8b5cf6 100%);
  color: #ffffff;
  box-shadow: 0 10rpx 32rpx rgba(236, 72, 153, 0.4), 0 4rpx 12rpx rgba(168, 85, 247, 0.2);
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.15);
}

.btn-export:active {
  transform: scale(0.97);
  box-shadow: 0 6rpx 20rpx rgba(236, 72, 153, 0.45);
}

.btn-color-picker {
  background: linear-gradient(135deg, #10b981 0%, #34d399 30%, #22d3ee 70%, #06b6d4 100%);
  color: #ffffff;
  box-shadow: 0 10rpx 32rpx rgba(16, 185, 129, 0.4), 0 4rpx 12rpx rgba(6, 182, 212, 0.2);
  text-shadow: 0 2rpx 4rpx rgba(0, 0, 0, 0.15);
}

.btn-color-picker:active {
  transform: scale(0.97);
  box-shadow: 0 6rpx 20rpx rgba(16, 185, 129, 0.45);
}

.btn-color-picker.disabled {
  background: #e5e7eb !important;
  color: #9ca3af !important;
  box-shadow: none !important;
  pointer-events: none;
  cursor: not-allowed;
}

.color-drawer-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  animation: fadeIn 0.2s ease;
}

.color-drawer {
  position: fixed;
  top: 0;
  right: 0;
  width: 85%;
  max-width: 600rpx;
  height: 100%;
  background: #ffffff;
  z-index: 1001;
  transform: translateX(100%);
  transition: transform 0.3s ease;
  display: flex;
  flex-direction: column;
}

.color-drawer.open {
  transform: translateX(0);
}

.color-drawer-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 28rpx 32rpx;
  padding-top: calc(28rpx + env(safe-area-inset-top));
  border-bottom: 1px solid #f0f0f0;
  background: linear-gradient(135deg, #ec4899 0%, #8b5cf6 100%);
}

.color-drawer-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #ffffff;
}

.color-drawer-close {
  width: 60rpx;
  height: 60rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48rpx;
  color: #ffffff;
  font-weight: 300;
}

.color-drawer-content {
  flex: 1;
  padding: 20rpx;
  overflow-y: auto;
}

.color-drawer-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10rpx;
}

.color-drawer-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10rpx;
  padding: 20rpx;
  border-radius: 16rpx;
  background: #f8f9fa;
  transition: all 0.2s;
  cursor: pointer;
  pointer-events: auto;
  position: relative;
  z-index: 1;
}

.color-drawer-item:active {
  background: linear-gradient(135deg, #e9ecef 0%, #dee2e6 100%);
  transform: scale(0.95);
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.1) inset;
}

.color-drawer-swatch {
  width: 80rpx;
  height: 80rpx;
  border-radius: 12rpx;
  border: 2rpx solid #ddd;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.1);
}

.color-drawer-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4rpx;
}

.color-drawer-code {
  font-size: 24rpx;
  font-weight: bold;
  color: #333;
}

.color-drawer-name {
  font-size: 20rpx;
  color: #999;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 100%;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
