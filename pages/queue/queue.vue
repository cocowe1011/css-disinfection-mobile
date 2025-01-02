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

<script>
import PalletList from '@/components/pallet-list.vue'
import QueueSelector from '@/components/queue-selector.vue'
import ScanButton from '@/components/scan-button.vue'
import request from '@/config/request'

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
      isReady: false,
      currentOrder: null
    }
  },
  async onLoad(options) {
    this.areaId = options.queueId
    this.areaName = options.name
    
    // 获取当前运行的订单
    await this.getCurrentOrder()
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
        const scanResult = await uni.scanCode({
          scanType: ['barCode'],  // 只支持条形码
          onlyFromCamera: true    // 只允许从相机扫码
        })
        
        console.log('扫码原始结果:', scanResult)
        
        // 检查是否有当前订单
        if (!this.currentOrder?.orderId) {
          uni.showToast({
            title: '请先选择订单',
            icon: 'none'
          })
          return
        }
        
        // 获取实际的扫码结果
        const scanData = Array.isArray(scanResult) ? scanResult[1] : scanResult
        const scanCode = scanData?.result
        
        console.log('处理后的扫码结果:', scanCode)
        
        if (!scanCode) {
          uni.showToast({
            title: '无效的扫码结果',
            icon: 'none'
          })
          return
        }
        
        // 构建新的托盘信息
        const newTray = {
          trayCode: scanCode,
          trayTime: new Date().toISOString().replace('T', ' ').split('.')[0],
          batchId: this.currentOrder.orderId
        }
        
        console.log('新托盘信息:', newTray)
        
        // 添加到现有托盘列表
        const updatedTrayInfo = this.palletList.map(tray => ({
          trayCode: tray.code || tray.trayCode,
          trayTime: tray.createTime || tray.trayTime,
          batchId: tray.batchId || ''
        }))
        
        // 添加新托盘
        updatedTrayInfo.push(newTray)
        
        console.log('更新的托盘列表:', updatedTrayInfo)
        
        // 更新队列信息
        const success = await this.updateQueueInfo(updatedTrayInfo)
        
        if (success) {
          uni.showToast({
            title: '添加成功',
            icon: 'success'
          })
        }
      } catch (error) {
        console.error('扫码错误:', error)
        uni.showToast({
          title: '扫描失败',
          icon: 'error'
        })
      } finally {
        this.scanning = false
      }
    },
    async handleDelete(pallet) {
      // 过滤掉要删除的托盘
      const updatedTrayInfo = this.palletList
        .filter(p => p.id !== pallet.id)  // 先过滤掉要删除的托盘
        .map(tray => ({                   // 然后转换格式
          trayCode: tray.code || tray.trayCode,
          trayTime: tray.createTime || tray.trayTime,
          batchId: tray.batchId || ''
        }))
      
      // 更新队列信息
      const success = await this.updateQueueInfo(updatedTrayInfo)
      
      if (success) {
        uni.showToast({
          title: '删除成功',
          icon: 'success'
        })
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
        console.log('请求参数:', { id: this.areaId })
        // 直接拼接在URL中传递参数
        const res = await request.get(`/queue_info/getQueueInfoById?id=${this.areaId}`)
        
        console.log('返回结果:', JSON.stringify(res))
        if (res.code === '200' && res.data) {
          let trayInfo = []
          try {
            trayInfo = res.data.trayInfo ? JSON.parse(res.data.trayInfo) : []
          } catch (e) {
            console.error('解析托盘信息失败:', e)
          }
          
          // 转换托盘数据格式
          this.palletList = trayInfo.map(tray => ({
            id: tray.trayCode, // 使用托盘编号作为唯一标识
            code: tray.trayCode,
            createTime: tray.trayTime,
            batchId: tray.batchId
          }))
        } else {
          this.palletList = []
          uni.showToast({
            title: res.message || '获取数据失败',
            icon: 'none'
          })
        }
      } catch (error) {
        console.error('获取托盘列表失败:', error)
        uni.showToast({
          title: '网络异常',
          icon: 'error'
        })
        this.palletList = []
      } finally {
        this.loading = false
      }
    },
    onPalletTap(item) {
      // 处理托盘点击事件
      console.log('托盘被点击：', item);
    },
    // 更新队列信息的公共方法
    async updateQueueInfo(trayInfo) {
      try {
        // 确保每个托盘都有必要的字段
        const formattedTrayInfo = trayInfo.map(tray => ({
          trayCode: tray.trayCode,
          trayTime: tray.trayTime,
          batchId: tray.batchId
        }))
        
        // 构建更新参数
        const params = {
          id: Number(this.areaId),
          trayInfo: JSON.stringify(formattedTrayInfo)
        }
        
        const res = await request.post('/queue_info/update', params)
        
        if (res.code === '200') {
          // 更新成功后重新获取最新数据
          await this.fetchPalletList()
          return true
        } else {
          uni.showToast({
            title: res.message || '操作失败',
            icon: 'none'
          })
          return false
        }
      } catch (error) {
        console.error('更新失败:', error)
        uni.showToast({
          title: '网络异常',
          icon: 'error'
        })
        return false
      }
    },
    // 获取当前运行的订单
    async getCurrentOrder() {
      try {
        const res = await request.post('/order_info/getNowRunningOrder')
        if (res.code === '200' && res.data) {
          this.currentOrder = res.data
        }
      } catch (error) {
        console.error('获取当前订单失败:', error)
      }
    }
  }
}
</script> 
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