<template>
  <view class="home-container">
    <!-- 状态栏占位 -->
    <view class="status-bar" :style="{ height: statusBarHeight + 'px' }"></view>
    
    <!-- 导航栏 -->
    <view class="header">
      <view class="welcome">欢迎回来，{{ username }}</view>
      <view class="logout" @tap="handleLogout">退出登录</view>
    </view>
    
    <view class="content">
      <!-- 订单信息卡片 -->
      <order-card :order="currentOrder"></order-card>
      
      <!-- 队列区域列表 -->
      <view class="queue-areas">
        <view class="section-title">队列区域</view>
        <view class="area-grid">
          <view 
            class="area-item" 
            v-for="area in queueAreas" 
            :key="area.id"
            @tap="navigateToQueue(area)"
          >
            <view class="area-content">
              <text class="area-name">{{ area.name }}</text>
              <text class="pallet-count">托盘数：{{ area.palletCount }}</text>
            </view>
            <view class="area-decoration">
              <text class="area-id">{{ area.id.toString().padStart(2, '0') }}</text>
              <text class="iconfont icon-right"></text>
            </view>
          </view>
        </view>
      </view>
    </view>
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
        inPort: '1号进货口',
        outPort: '2号出货口',
        status: '进行中',
      },
      queueAreas: [
        { id: 1, name: 'A区域', palletCount: 5 },
        { id: 2, name: 'B区域', palletCount: 3 },
        { id: 3, name: 'C区域', palletCount: 7 }
      ],
      statusBarHeight: 0,
    }
  },
  onLoad() {
    // 获取状态栏高度
    const systemInfo = uni.getSystemInfoSync()
    this.statusBarHeight = systemInfo.statusBarHeight
  },
  methods: {
    handleLogout() {
      uni.clearStorageSync()
      uni.reLaunch({
        url: '/pages/login/login'
      })
    },
    navigateToQueue(area) {
      uni.navigateTo({
        url: `/pages/queue/queue?id=${area.id}&name=${area.name}`
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
    background: linear-gradient(90deg, #1a2a6c, #b21f1f);
    padding: 0 30rpx;
    padding-top: calc(var(--status-bar-height) + 20rpx);
    padding-bottom: 40rpx;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #fff;
    
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
  
  .content {
    padding: 30rpx;
    
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