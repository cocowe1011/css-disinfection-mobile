<template>
  <view class="queue-container">
    <!-- 状态栏占位 -->
    <view class="status-bar" :style="{ height: statusBarHeight + 'px' }"></view>
    
    <!-- 导航栏 -->
    <view class="nav-bar">
      <view class="nav-content">
        <view class="area-info">
          <text class="area-name">{{ areaName }}</text>
          <text class="pallet-count">托盘数量：{{ palletList.length }}</text>
        </view>
        <scan-button 
          :loading="scanning" 
          @scan="handleScan"
          class="scan-btn"
        ></scan-button>
      </view>
    </view>
    
    <!-- 主内容区域 -->
    <view class="main-content">
      <pull-to-refresh 
        v-model="refreshing"
        @refresh="handleRefresh"
      >
        <pallet-list 
          :pallets="palletList"
          @delete="handleDelete"
          @move="handleMove"
          @batchMove="handleBatchMove"
          @batchDelete="handleBatchDelete"
          :loading="loading"
        ></pallet-list>
      </pull-to-refresh>
    </view>
    
    <!-- 恢复队列选择器 -->
    <queue-selector
      :visible="showQueueSelector"
      :current-queue-id="areaId"
      @close="showQueueSelector = false"
      @select="handleQueueSelect"
    />
  </view>
</template>

<script>
import PalletList from '@/components/pallet-list.vue'
import QueueSelector from '@/components/queue-selector.vue'
import ScanButton from '@/components/scan-button.vue'
import PullToRefresh from '@/components/pull-to-refresh.vue'

export default {
  components: {
    PalletList,
    QueueSelector,
    ScanButton,
    PullToRefresh
  },
  data() {
    return {
      areaId: '',
      areaName: '',
      palletList: [],
      showQueueSelector: false,
      currentPallet: null,
      scanning: false,
      loading: false,
      refreshing: false,
      statusBarHeight: 0
    }
  },
  onLoad(options) {
    this.areaId = options.id
    this.areaName = options.name
    // 获取状态栏高度
    const systemInfo = uni.getSystemInfoSync()
    this.statusBarHeight = systemInfo.statusBarHeight
    
    // 模拟获取托盘数据
    this.palletList = [
      { id: 1, code: 'P001', createTime: '2024-03-20 10:00' },
      { id: 2, code: 'P002', createTime: '2024-03-20 11:00' }
    ]
  },
  methods: {
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
          title: '扫码失败',
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
            title: `已移动到${queue.name}`,
            icon: 'success'
          })
        }
        this.currentPallet = null
        this.showQueueSelector = false
      }
    },
    async handleRefresh() {
      try {
        // 模拟刷新数据
        await new Promise(resolve => setTimeout(resolve, 1000))
        this.palletList = [
          { id: 1, code: 'P001', createTime: '2024-03-20 10:00' },
          { id: 2, code: 'P002', createTime: '2024-03-20 11:00' }
        ]
      } finally {
        this.refreshing = false
      }
    },
    handleBatchMove(ids) {
      this.currentPallet = { ids }
      this.showQueueSelector = true
    },
    handleBatchDelete(ids) {
      this.palletList = this.palletList.filter(p => !ids.includes(p.id))
      uni.showToast({
        title: '删除成功',
        icon: 'success'
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.queue-container {
  min-height: 100vh;
  background: $bg-light;
  
  .status-bar {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background: #fff;
    z-index: 101;
  }
  
  .nav-bar {
    position: fixed;
    top: var(--status-bar-height);
    left: 0;
    right: 0;
    height: 88rpx;
    background: #fff;
    z-index: 100;
    box-shadow: $card-shadow;
    
    .nav-content {
      height: 100%;
      padding: 0 30rpx;
      display: flex;
      justify-content: space-between;
      align-items: center;
      
      .area-info {
        display: flex;
        align-items: center;
        line-height: 1.4;
        
        .area-name {
          font-size: 32rpx;
          color: $text-primary;
          font-weight: bold;
        }
        
        .pallet-count {
          font-size: 26rpx;
          color: $text-secondary;
          margin-left: 20rpx;
          background: $bg-light;
          padding: 4rpx 16rpx;
          border-radius: 20rpx;
        }
      }
      
      .scan-btn {
        height: 64rpx;
      }
    }
  }
  
  .main-content {
    padding: 30rpx;
    padding-top: calc(var(--status-bar-height) + 88rpx + 30rpx);
  }
}
</style> 