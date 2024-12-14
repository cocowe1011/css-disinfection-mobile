<template>
  <view class="order-card" :class="{ loading }">
    <view class="card-content" v-if="!loading">
      <!-- 左侧状态条 -->
      <view class="status-bar"></view>
      
      <!-- 主要内容 -->
      <view class="main-content">
        <!-- 顶部信息 -->
        <view class="top-info">
          <view class="product-info">
            <text class="product-name">{{ order.productName }}</text>
            <view class="batch-info">
              <text class="label">批次号</text>
              <text class="batch-id">{{ order.batchId }}</text>
            </view>
          </view>
          <view class="status-info">
            <text class="iconfont icon-processing"></text>
            <text class="status-text">进行中</text>
          </view>
        </view>
        
        <!-- 详细信息 -->
        <view class="detail-info">
          <view class="info-section">
            <view class="section-title">生产设备</view>
            <view class="section-content">
              <view class="info-item">
                <text class="iconfont icon-preheat"></text>
                <view class="item-content">
                  <text class="label">预热房</text>
                  <text class="value">{{ order.preheatingRoom }}</text>
                </view>
              </view>
              <view class="info-item">
                <text class="iconfont icon-sterilizer"></text>
                <view class="item-content">
                  <text class="label">灭菌柜</text>
                  <text class="value">{{ order.sterilizer }}</text>
                </view>
              </view>
            </view>
          </view>
          
          <view class="info-section">
            <view class="section-title">出入口</view>
            <view class="section-content">
              <view class="info-item">
                <text class="iconfont icon-in"></text>
                <view class="item-content">
                  <text class="label">进货口</text>
                  <text class="value">{{ order.inPort }}</text>
                </view>
              </view>
              <view class="info-item">
                <text class="iconfont icon-out"></text>
                <view class="item-content">
                  <text class="label">出货口</text>
                  <text class="value">{{ order.outPort }}</text>
                </view>
              </view>
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
          <view class="skeleton-row" v-for="i in 4" :key="i"></view>
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
      background: linear-gradient(to bottom, $success-color, adjust-hue($success-color, 20deg));
    }
    
    .main-content {
      flex: 1;
      padding: 30rpx;
      
      .top-info {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        margin-bottom: 30rpx;
        padding-bottom: 20rpx;
        border-bottom: 1px solid $border-color;
        
        .product-info {
          .product-name {
            font-size: 36rpx;
            color: $text-primary;
            font-weight: bold;
            margin-bottom: 12rpx;
          }
          
          .batch-info {
            display: flex;
            align-items: center;
            
            .label {
              font-size: 24rpx;
              color: $text-secondary;
              margin-right: 10rpx;
            }
            
            .batch-id {
              font-size: 26rpx;
              color: $primary-color;
              background: rgba($primary-color, 0.1);
              padding: 4rpx 16rpx;
              border-radius: 20rpx;
            }
          }
        }
        
        .status-info {
          display: flex;
          align-items: center;
          padding: 8rpx 20rpx;
          border-radius: 30rpx;
          background: rgba($success-color, 0.1);
          color: $success-color;
          
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
        .info-section {
          margin-bottom: 24rpx;
          
          &:last-child {
            margin-bottom: 0;
          }
          
          .section-title {
            font-size: 24rpx;
            color: $text-secondary;
            margin-bottom: 16rpx;
            padding-left: 16rpx;
            border-left: 4rpx solid $primary-color;
          }
          
          .section-content {
            display: flex;
            margin: -10rpx;
            
            .info-item {
              flex: 1;
              display: flex;
              align-items: center;
              margin: 10rpx;
              padding: 20rpx;
              background: #e5eaf2;
              border-radius: $border-radius;
              transition: all $transition-fast;
              box-shadow: 0 4rpx 12rpx rgba(26, 42, 108, 0.06);
              
              &:hover {
                background: #dce3ed;
                box-shadow: 0 6rpx 16rpx rgba(26, 42, 108, 0.1);
              }
              
              .iconfont {
                font-size: 40rpx;
                color: $primary-color;
                margin-right: 16rpx;
              }
              
              .item-content {
                flex: 1;
                text-align: left;
                
                .label {
                  font-size: 24rpx;
                  color: $text-secondary;
                  margin-bottom: 6rpx;
                  display: block;
                }
                
                .value {
                  font-size: 28rpx;
                  color: $text-primary;
                  font-weight: 500;
                  line-height: 1.4;
                }
              }
            }
          }
        }
      }
    }
  }
  
  // 骨架屏样式...
}
</style> 