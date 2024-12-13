<template>
  <view class="order-card" :class="{ loading }">
    <view class="card-content" v-if="!loading">
      <!-- 左侧状态条 -->
      <view class="status-bar" :class="order.status"></view>
      
      <!-- 主要内容 -->
      <view class="main-content">
        <!-- 顶部信息 -->
        <view class="top-info">
          <view class="product-info">
            <text class="product-name">{{ order.productName }}</text>
            <text class="batch-id">{{ order.batchId }}</text>
          </view>
          <view class="status-info" :class="order.status">
            <text class="iconfont" :class="statusIcon"></text>
            <text class="status-text">{{ order.status }}</text>
          </view>
        </view>
        
        <!-- 详细信息 -->
        <view class="detail-info">
          <view class="info-row">
            <view class="info-col">
              <text class="iconfont icon-preheat"></text>
              <text class="label">预热房</text>
              <text class="value">{{ order.preheatingRoom }}</text>
            </view>
            <view class="info-col">
              <text class="iconfont icon-in"></text>
              <text class="label">进货口</text>
              <text class="value">{{ order.inPort }}</text>
            </view>
            <view class="info-col">
              <text class="iconfont icon-out"></text>
              <text class="label">出货口</text>
              <text class="value">{{ order.outPort }}</text>
            </view>
          </view>
        </view>
      </view>
    </view>
    
    <!-- 骨架屏 -->
    <view class="skeleton" v-else>
      <view class="skeleton-bar"></view>
      <view class="skeleton-content">
        <view class="skeleton-header">
          <view class="skeleton-title"></view>
          <view class="skeleton-tag"></view>
        </view>
        <view class="skeleton-info">
          <view class="skeleton-row" v-for="i in 3" :key="i"></view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  name: 'OrderCard',
  props: {
    order: {
      type: Object,
      required: true
    },
    loading: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    statusIcon() {
      const icons = {
        '进行中': 'icon-processing',
        '待处理': 'icon-pending',
        '已完成': 'icon-completed'
      }
      return icons[this.order.status] || 'icon-default'
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/styles/variables.scss';

.order-card {
  background: #fff;
  border-radius: $border-radius;
  margin-bottom: 30rpx;
  box-shadow: $card-shadow;
  overflow: hidden;
  transition: all $transition-fast;
  
  &:active {
    transform: translateY(2rpx);
    box-shadow: $hover-shadow;
  }
  
  .card-content {
    display: flex;
    min-height: 240rpx;
    
    .status-bar {
      width: 12rpx;
      background: $primary-gradient;
      
      &.进行中 {
        background: linear-gradient(to bottom, $success-color, adjust-hue($success-color, 20deg));
      }
      
      &.待处理 {
        background: linear-gradient(to bottom, $warning-color, adjust-hue($warning-color, 20deg));
      }
    }
    
    .main-content {
      flex: 1;
      padding: 30rpx;
      
      .top-info {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        margin-bottom: 30rpx;
        
        .product-info {
          .product-name {
            font-size: 34rpx;
            color: $text-primary;
            font-weight: bold;
            margin-bottom: 12rpx;
            display: block;
          }
          
          .batch-id {
            font-size: 24rpx;
            color: $text-secondary;
            background: $bg-light;
            padding: 4rpx 16rpx;
            border-radius: 20rpx;
          }
        }
        
        .status-info {
          display: flex;
          align-items: center;
          padding: 8rpx 20rpx;
          border-radius: 30rpx;
          background: rgba($success-color, 0.1);
          
          &.进行中 {
            background: rgba($success-color, 0.1);
            color: $success-color;
          }
          
          &.待处理 {
            background: rgba($warning-color, 0.1);
            color: $warning-color;
          }
          
          .iconfont {
            font-size: 28rpx;
            margin-right: 8rpx;
          }
          
          .status-text {
            font-size: 26rpx;
            font-weight: 500;
          }
        }
      }
      
      .detail-info {
        .info-row {
          display: flex;
          justify-content: space-between;
          gap: 20rpx;
          
          .info-col {
            flex: 1;
            text-align: center;
            padding: 20rpx;
            background: $bg-light;
            border-radius: $border-radius;
            
            .iconfont {
              font-size: 40rpx;
              color: $text-secondary;
              margin-bottom: 12rpx;
              display: block;
            }
            
            .label {
              font-size: 24rpx;
              color: $text-secondary;
              margin-bottom: 8rpx;
              display: block;
            }
            
            .value {
              font-size: 28rpx;
              color: $text-primary;
              font-weight: 500;
            }
          }
        }
      }
    }
  }
  
  // 骨架屏样式...
}
</style> 