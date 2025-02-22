<template>
  <view class="queue-selector" v-if="visible">
    <view class="mask" @tap="handleClose"></view>
    <view class="content">
      <view class="header">
        <text class="title">选择目标队列</text>
        <text class="close" @tap="handleClose">×</text>
      </view>
      
      <scroll-view scroll-y class="queue-list">
        <view class="queue-grid">
          <view 
            class="queue-item"
            v-for="queue in queues"
            :key="queue.id"
            @tap="handleSelect(queue)"
          >
            <text class="queue-name">{{ queue.name }}</text>
            <text class="count">{{ queue.count }}托盘</text>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  name: 'QueueSelector',
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    currentQueueId: {
      type: [Number, String],
      default: ''
    }
  },
  data() {
    return {
      screenWidth: 375,
      queues: [
        { id: 1, name: '上货区', count: 5 },
        { id: 2, name: '小车1区', count: 3 },
        { id: 3, name: 'A1', count: 7 },
        { id: 4, name: 'B1', count: 2 },
        { id: 5, name: 'C1', count: 4 },
        { id: 6, name: 'D1', count: 6 },
        { id: 7, name: 'E1', count: 3 },
        { id: 8, name: 'F1', count: 5 },
        { id: 9, name: 'G1', count: 4 },
        { id: 10, name: 'A2', count: 6 },
        { id: 11, name: 'B2', count: 3 },
        { id: 12, name: 'C2', count: 5 },
		{ id: 13, name: 'D2', count: 5 },
		{ id: 14, name: 'E2', count: 5 },
      ]
    }
  },
  mounted() {
    const systemInfo = uni.getSystemInfoSync()
    this.screenWidth = systemInfo.windowWidth
  },
  computed: {
    availableQueues() {
      return this.queues.filter(q => q.id !== this.currentQueueId)
    }
  },
  methods: {
    handleClose() {
      this.$emit('close')
    },
    handleSelect(queue) {
      this.$emit('select', queue)
      this.handleClose()
    }
  }
}
</script>

<style lang="scss" scoped>
.queue-selector {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 999;
  
  .mask {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.4);
  }
  
  .content {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    background: #fff;
    border-radius: 24rpx 24rpx 0 0;
    max-height: 65vh;
    
    .header {
      padding: 20rpx 30rpx;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 1px solid #f0f0f0;
      
      .title {
        font-size: 32rpx;
        color: $text-primary;
        font-weight: bold;
      }
      
      .close {
        font-size: 40rpx;
        color: $text-secondary;
        padding: 10rpx;
        margin: -10rpx;
      }
    }
    
    .queue-list {
      max-height: calc(65vh - 80rpx);
      padding: 20rpx;
      box-sizing: border-box;
      
      .queue-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 16rpx;
        
        .queue-item {
          padding: 20rpx;
          background: linear-gradient(135deg, #f6f7f9, #f0f2f5);
          border-radius: 12rpx;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          transition: all 0.2s ease;
          min-width: 0;
          border: 1px solid rgba($primary-color, 0.1);
          
          .queue-name {
            font-size: 30rpx;
            color: $primary-color;
            font-weight: 500;
            margin-bottom: 8rpx;
            width: 100%;
            text-align: center;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            padding: 0 10rpx;
            box-sizing: border-box;
          }
          
          .count {
            font-size: 24rpx;
            color: rgba($primary-color, 0.7);
          }
          
          &:active {
            background: linear-gradient(135deg, rgba($primary-color, 0.08), rgba($primary-color, 0.12));
            border-color: rgba($primary-color, 0.2);
            transform: scale(0.98);
          }
        }
      }
    }
  }
}

/* #ifdef H5 */
@media screen and (min-width: 768px) {
  .queue-selector {
    .queue-list {
      padding: 20rpx 30rpx;
      
      .queue-grid {
        grid-template-columns: repeat(3, 1fr);
        margin: 0;
      }
    }
  }
}
/* #endif */
</style> 