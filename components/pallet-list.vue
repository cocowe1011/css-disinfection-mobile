<template>
  <view class="pallet-list">
    <view 
      class="pallet-item" 
      v-for="pallet in pallets" 
      :key="pallet.id"
      :class="{ 'is-moving': movingId === pallet.id }"
      @tap="handleTap(pallet)"
    >
      <view class="area-content">
        <text class="pallet-code">{{ pallet.code }}</text>
        <text class="create-time">{{ pallet.createTime }}</text>
      </view>
      
      <view class="actions">
        <button 
          class="action-btn move" 
          @tap.stop="handleMove(pallet)"
          :disabled="movingId === pallet.id"
        >
          <text class="iconfont icon-move"></text>
          <text class="btn-text">移动</text>
        </button>
        <button 
          class="action-btn delete" 
          @tap.stop="handleDelete(pallet)"
          :disabled="movingId === pallet.id"
        >
          <text class="iconfont icon-delete"></text>
          <text class="btn-text">删除</text>
        </button>
      </view>
    </view>
    
    <view class="empty" v-if="pallets.length === 0">
      <text class="iconfont icon-empty"></text>
      <text class="empty-text">暂无托盘数据</text>
    </view>
  </view>
</template>

<script>
export default {
  name: 'PalletList',
  props: {
    pallets: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      movingId: null
    }
  },
  methods: {
    handleMove(pallet) {
      this.movingId = pallet.id
      this.$emit('move', pallet)
    },
    handleDelete(pallet) {
      uni.showModal({
        title: '确认删除',
        content: '是否确认删除该托盘？',
        success: (res) => {
          if (res.confirm) {
            this.$emit('delete', pallet)
          }
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/styles/variables.scss';

.pallet-list {
  .pallet-item {
    background: #fff;
    padding: 30rpx;
    border-radius: $border-radius;
    margin-bottom: 20rpx;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: $card-shadow;
    transition: all $transition-fast;
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
    }
    
    &.is-moving {
      opacity: 0.7;
      
      &::after {
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: rgba(255,255,255,0.6);
        z-index: 1;
      }
    }
    
    .area-content {
      flex: 1;
      margin-left: 20rpx;
      
      .pallet-code {
        font-size: 32rpx;
        color: $text-primary;
        font-weight: bold;
        margin-bottom: 8rpx;
        display: block;
      }
      
      .create-time {
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
    
    .actions {
      display: flex;
      gap: 20rpx;
      
      .action-btn {
        display: flex;
        align-items: center;
        padding: 12rpx 24rpx;
        border-radius: 30rpx;
        border: none;
        transition: all $transition-fast;
        
        .iconfont {
          font-size: 28rpx;
        }
        
        &.move {
          background: rgba(24, 144, 255, 0.1);
          color: #1890ff;
          
          &:active {
            background: rgba(24, 144, 255, 0.2);
          }
        }
        
        &.delete {
          background: rgba(245, 34, 45, 0.1);
          color: #f5222d;
          
          &:active {
            background: rgba(245, 34, 45, 0.2);
          }
        }
        
        &[disabled] {
          opacity: 0.5;
          pointer-events: none;
        }
        
        .btn-text {
          font-size: 26rpx;
          margin-left: 8rpx;
        }
      }
    }
  }
  
  .empty {
    text-align: center;
    padding: 100rpx 0;
    color: $text-light;
    
    .iconfont {
      font-size: 80rpx;
      margin-bottom: 20rpx;
      display: block;
    }
    
    .empty-text {
      font-size: 28rpx;
    }
  }
}
</style> 