<template>
  <view class="queue-container">
    <!-- 导航栏 -->
    <uni-nav-bar
      fixed
      status-bar
      left-icon="left"
      :border="false"
      title="队列详情"
      background-color="#1a2a6c"
      color="#FFFFFF"
      @clickLeft="goBack"
    />
    
    <!-- 固定区域 -->
    <view class="fixed-section">
      <!-- 区域信息卡片 -->
      <view class="area-card">
        <view class="area-info">
          <text class="area-name">{{ areaName }}</text>
          <text class="pallet-count">托盘数量：{{ palletList.length }}</text>
        </view>
        <button class="scan-btn" :class="{ loading: scanning }" @tap="handleScan">
          <text class="iconfont icon-scan"></text>
          <text class="btn-text">扫码添加</text>
        </button>
      </view>
    </view>
    
    <!-- 可滚动区域 -->
    <scroll-view 
      class="scroll-section"
      scroll-y
    >
      <view class="main-content">
        <view v-if="loading" class="loading-wrapper">
          <view class="loading-icon"></view>
          <text class="loading-text">加载中...</text>
        </view>
        <pallet-list 
          v-else
          :pallets="palletList"
          @delete="handleDelete"
          @move="handleMove"
          :loading="loading"
          @tap="onPalletTap"
        ></pallet-list>
      </view>
    </scroll-view>
    
    <!-- 队列选择器 -->
    <queue-selector
      :visible="showQueueSelector"
      :current-queue-id="areaId"
      @close="showQueueSelector = false"
      @select="handleQueueSelect"
    />
  </view>
</template>

<style lang="scss" scoped>
.queue-container {
  min-height: 100vh;
  background: $bg-light;
  padding-top: 0;
  display: flex;
  flex-direction: column;
  
  :deep(.uni-nav-bar) {
    /* #ifdef APP-PLUS */
    padding-top: var(--status-bar-height);
    /* #endif */
  }
  
  :deep(.uni-nav-bar-fixed) {
    background: linear-gradient(90deg, #1a2a6c, #b21f1f);
  }
  
  :deep(.uni-navbar__header) {
    background: transparent !important;
  }
  
  :deep(.uni-navbar__header-container) {
    background: transparent !important;
  }
  
  :deep(.uni-nav-bar-text) {
    color: #fff;
    font-size: 32rpx;
    font-weight: bold;
  }
  
  :deep(.uni-nav-bar__btn-icon) {
    color: #fff !important;
    font-size: 36rpx;
  }
  
  .fixed-section {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1;
    background: $bg-light;
    margin-top: 44px;
    /* #ifdef APP-PLUS */
    margin-top: calc(44px + var(--status-bar-height));
    /* #endif */
  }
  
  .area-card {
    margin: 20rpx;
    padding: 24rpx 30rpx;
    background: #fff;
    border-radius: $border-radius;
    background: linear-gradient(135deg, #1a2a6c, #4286f4);
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: relative;
    overflow: hidden;
    
    &::after {
      content: '';
      position: absolute;
      right: -60rpx;
      top: -60rpx;
      width: 200rpx;
      height: 200rpx;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 50%;
      z-index: 0;
    }
    
    .area-info {
      display: flex;
      flex-direction: column;
      gap: 8rpx;
      z-index: 1;
      
      .area-name {
        font-size: 34rpx;
        color: #fff;
        font-weight: 600;
      }
      
      .pallet-count {
        font-size: 26rpx;
        color: rgba(255, 255, 255, 0.9);
        background: rgba(255, 255, 255, 0.15);
        padding: 4rpx 16rpx;
        border-radius: 20rpx;
        display: inline-block;
      }
    }
    
    .scan-btn {
      height: 72rpx;
      padding: 0 30rpx;
      background: #fff;
      border: none;
      border-radius: 36rpx;
      color: #fff;
      font-size: 28rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      box-shadow: 0 6rpx 16rpx rgba(0, 0, 0, 0.1);
      transition: all 0.3s ease;
      z-index: 1;
      margin-right: -10rpx;
      
      .iconfont {
        font-size: 32rpx;
        margin-right: 8rpx;
        color: #1a2a6c;
        transition: transform 0.3s ease;
      }
      
      .btn-text {
        color: #1a2a6c;
        font-weight: 500;
      }
      
      &.loading {
        opacity: 0.8;
        .iconfont {
          animation: rotate 1s linear infinite;
        }
      }
      
      &:active {
        transform: translateY(2rpx);
        box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
        
        .iconfont {
          transform: scale(0.95);
        }
      }
    }
  }
  
  .scroll-section {
    position: fixed;
    top: calc(140rpx + 50px);
    /* #ifdef APP-PLUS */
    top: calc(140rpx + 50px + var(--status-bar-height));
    /* #endif */
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 0;
    background: $bg-light;
  }
  
  .main-content {
    padding: 30rpx;
  }
  
  .loading-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 60rpx 0;
    
    .loading-icon {
      width: 60rpx;
      height: 60rpx;
      border: 4rpx solid #f3f3f3;
      border-top: 4rpx solid $primary-color;
      border-radius: 50%;
      animation: spin 1s linear infinite;
      margin-bottom: 20rpx;
    }
    
    .loading-text {
      font-size: 28rpx;
      color: $text-secondary;
    }
  }
  
  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>

<script>
import PalletList from '@/components/pallet-list.vue'
import QueueSelector from '@/components/queue-selector.vue'
import ScanButton from '@/components/scan-button.vue'

export default {
  components: {
    PalletList,
    QueueSelector,
    ScanButton
  },
  data() {
    return {
      areaId: '',
      areaName: '',
      palletList: [],
      showQueueSelector: false,
      currentPallet: null,
      scanning: false,
      loading: true,
      navBgColor: '#2B32B2',
      navTextColor: '#FFFFFF',
      pageReady: false,
      isLoaded: false,
      isReady: false
    }
  },
  async onLoad(options) {
    this.areaId = options.id
    this.areaName = options.name
    
    // 获取托盘数据
    await this.fetchPalletList()
  },
  methods: {
    goBack() {
      uni.navigateBack()
    },
    async handleScan() {
      if (this.scanning) return
      this.scanning = true
      
      try {
        const res = await uni.scanCode()
        console.log('扫码结果：', res)
        
        // 这里添加托盘
        this.palletList.push({
          id: Date.now(),
          code: res.result,
          createTime: new Date().toLocaleString()
        })
        
        uni.showToast({
          title: '添加成功',
          icon: 'success'
        })
      } catch (error) {
        uni.showToast({
          title: '扫描失败',
          icon: 'error'
        })
      } finally {
        this.scanning = false
      }
    },
    handleDelete(pallet) {
      const index = this.palletList.findIndex(p => p.id === pallet.id)
      if (index > -1) {
        this.palletList.splice(index, 1)
      }
    },
    handleMove(pallet) {
      this.currentPallet = pallet
      this.showQueueSelector = true
    },
    handleQueueSelect(queue) {
      if (this.currentPallet) {
        const index = this.palletList.findIndex(p => p.id === this.currentPallet.id)
        if (index > -1) {
          this.palletList.splice(index, 1)
          uni.showToast({
            title: `移动到${queue.name}`,
            icon: 'success'
          })
        }
        this.currentPallet = null
        this.showQueueSelector = false
      }
    },
    async fetchPalletList() {
      try {
        // 这里是实际的数据获取逻辑
        // 模拟异步请求
        await new Promise(resolve => setTimeout(resolve, 500))
        this.palletList = [
          { id: 1, code: 'P001', createTime: '2024-03-20 10:00' },
          { id: 2, code: 'P002', createTime: '2024-03-20 10:15' },
          // ... 其他数据
        ]
      } finally {
        this.loading = false
      }
    },
    onPalletTap(item) {
      // 处理托盘点击事件
      console.log('托盘被点击：', item);
    }
  }
}
</script> 