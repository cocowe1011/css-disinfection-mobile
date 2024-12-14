<template>
  <view class="home-container">
    <!-- 状态栏占位 -->
    <view class="status-bar" :style="{ height: statusBarHeight + 'px' }"></view>
    
    <!-- 导航栏 -->
    <view class="header">
      <view class="welcome">欢迎回来，{{ username }}</view>
      <view class="logout" @tap="handleLogout">退出登录</view>
    </view>
    
    <!-- 可滚动的内容区域 -->
    <scroll-view 
      class="scroll-content"
      scroll-y
      :style="{ opacity: pageReady ? 1 : 0 }"
    >
      <view class="content">
        <!-- 订单信息卡片 -->
        <order-card :order="currentOrder"></order-card>
        
        <!-- 队列区域列表 -->
        <view class="queue-areas">
          <view 
            class="queue-group" 
            v-for="group in queueGroups" 
            :key="group.title"
          >
            <view class="section-title">{{ group.title }}</view>
            <view class="area-grid">
              <view 
                class="area-item" 
                v-for="area in group.areas" 
                :key="area.id"
                @tap="navigateToQueue(area)"
              >
                <view class="area-content">
                  <text class="area-name">{{ area.name }}</text>
                  <text class="pallet-count">托盘数：{{ area.palletCount }}</text>
                </view>
                <view class="area-decoration">
                  <text class="area-id">{{ area.id }}</text>
                  <text class="iconfont icon-right"></text>
                </view>
              </view>
            </view>
          </view>
        </view>
      </view>
    </scroll-view>
  </view>
</template>

<script>
import OrderCard from '@/components/order-card.vue'

export default {
  components: {
    OrderCard
  },
  data() {
    return {
      username: '张工',
      currentOrder: {
        batchId: 'B2024032001',
        productName: '消毒液',
        preheatingRoom: 'A区预热房',
        sterilizer: '1号灭菌柜',
        inPort: '1号进货口',
        outPort: '2号出货口',
        status: '进行中',
      },
      queueGroups: [
        {
          title: '物流区域',
          areas: [
            { id: 1, name: '上货区', palletCount: 8 },
            { id: 2, name: '下货区', palletCount: 5 },
            { id: 3, name: '缓冲区1', palletCount: 3 },
            { id: 4, name: '缓冲区2', palletCount: 4 },
            { id: 5, name: '缓冲区3', palletCount: 6 },
          ]
        },
        {
          title: '预热房区域1',
          areas: [
            { id: 6, name: '预热房A1', palletCount: 4 },
            { id: 7, name: '预热房B1', palletCount: 3 },
            { id: 8, name: '预热房C1', palletCount: 5 },
            { id: 9, name: '预热房D1', palletCount: 2 },
            { id: 10, name: '预热房E1', palletCount: 6 },
            { id: 11, name: '预热房F1', palletCount: 4 },
            { id: 12, name: '预热房G1', palletCount: 3 },
          ]
        },
        {
          title: '预热房区域2',
          areas: [
            { id: 13, name: '预热房A2', palletCount: 5 },
            { id: 14, name: '预热房B2', palletCount: 4 },
            { id: 15, name: '预热房C2', palletCount: 3 },
            { id: 16, name: '预热房D2', palletCount: 6 },
            { id: 17, name: '预热房E2', palletCount: 2 },
            { id: 18, name: '预热房F2', palletCount: 5 },
            { id: 19, name: '预热房G2', palletCount: 4 },
          ]
        },
        {
          title: '灭菌区域',
          areas: [
            { id: 20, name: '灭菌区1#', palletCount: 7 },
            { id: 21, name: '灭菌区2#', palletCount: 5 },
            { id: 22, name: '灭菌区3#', palletCount: 4 },
            { id: 23, name: '灭菌区4#', palletCount: 6 },
            { id: 24, name: '灭菌区5#', palletCount: 3 },
            { id: 25, name: '灭菌区6#', palletCount: 5 },
            { id: 26, name: '灭菌区7#', palletCount: 4 },
          ]
        }
      ],
      statusBarHeight: 0,
      pageReady: false
    }
  },
  async onLoad() {
    // 获取状态栏高度
    const systemInfo = uni.getSystemInfoSync()
    this.statusBarHeight = systemInfo.statusBarHeight
    
    // 模拟数据加载
    await new Promise(resolve => setTimeout(resolve, 100))
    this.pageReady = true
  },
  methods: {
    handleLogout() {
      uni.clearStorageSync()
      uni.reLaunch({
        url: '/pages/login/login'
      })
    },
    navigateToQueue(area) {
      uni.showLoading({
        title: '加载中...',
        mask: true
      })
      
      // 先准备好数据
      const palletList = [
        { id: 1, code: 'P001', createTime: '2024-03-20 10:00' },
        { id: 2, code: 'P002', createTime: '2024-03-20 10:15' },
        // ... 其他数据
      ]
      
      // 将数据存到全局状态或缓存中
      getApp().globalData = getApp().globalData || {}
      getApp().globalData.palletList = palletList
      
      uni.navigateTo({
        url: `/pages/queue/queue?id=${area.id}&name=${area.name}`,
        success: () => {
          uni.hideLoading()
        },
        fail: () => {
          uni.hideLoading()
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.home-container {
  min-height: 100vh;
  background: #f5f7fa;
  
  .status-bar {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background: linear-gradient(90deg, #1a2a6c, #b21f1f);
    z-index: 101;
  }
  
  .header {
    position: fixed;
    top: var(--status-bar-height);
    left: 0;
    right: 0;
    background: linear-gradient(90deg, #1a2a6c, #b21f1f);
    padding: 20rpx 30rpx;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #fff;
    z-index: 100;
    
    .welcome {
      font-size: 32rpx;
    }
    
    .logout {
      font-size: 28rpx;
      padding: 10rpx 20rpx;
      border: 1px solid rgba(255,255,255,0.5);
      border-radius: 30rpx;
    }
  }
  
  .scroll-content {
    position: fixed;
    top: calc(var(--status-bar-height) + 88rpx);
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1;
    transition: opacity 0.3s ease;
  }
  
  .content {
    padding: 30rpx;
    padding-bottom: env(safe-area-inset-bottom);  // 适配底部安全区域
    
    .section-title {
      font-size: 32rpx;
      color: #333;
      font-weight: bold;
      margin: 30rpx 0;
    }
    
    .area-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 20rpx;
      
      .area-item {
        background: #fff;
        padding: 30rpx;
        border-radius: $border-radius;
        box-shadow: $card-shadow;
        display: flex;
        justify-content: space-between;
        align-items: center;
        transition: all 0.3s ease;
        position: relative;
        overflow: hidden;
        
        &::before {
          content: '';
          position: absolute;
          top: 0;
          left: 0;
          width: 6rpx;
          height: 100%;
          background: linear-gradient(to bottom, #1a2a6c, #b21f1f);
          opacity: 0.8;
        }
        
        &:active {
          transform: scale(0.98);
          box-shadow: $hover-shadow;
        }
        
        .area-content {
          flex: 1;
          margin-left: 20rpx;
          
          .area-name {
            font-size: 32rpx;
            color: $text-primary;
            font-weight: bold;
            margin-bottom: 8rpx;
            display: block;
          }
          
          .pallet-count {
            font-size: 26rpx;
            color: $text-secondary;
            display: flex;
            align-items: center;
            
            &::before {
              content: '';
              width: 6rpx;
              height: 6rpx;
              background: #1a2a6c;
              border-radius: 50%;
              margin-right: 8rpx;
            }
          }
        }
        
        .area-decoration {
          text-align: right;
          
          .area-id {
            font-size: 40rpx;
            font-weight: bold;
            color: rgba(26, 42, 108, 0.1);
            display: block;
          }
          
          .iconfont {
            font-size: 32rpx;
            color: $text-secondary;
            margin-top: 8rpx;
          }
        }
      }
    }
  }
}
</style> 