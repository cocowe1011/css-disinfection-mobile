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
            <text class="count">托盘数：{{ queue.count }}</text>
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
      queues: [
        { id: 1, name: '1号队列', count: 5 },
        { id: 2, name: '2号队列', count: 3 },
        { id: 3, name: '3号队列', count: 7 },
        { id: 4, name: '4号队列', count: 2 },
        { id: 5, name: '5号队列', count: 4 },
        { id: 6, name: '6号队列', count: 6 },
        { id: 7, name: '7号队列', count: 3 },
        { id: 8, name: '8号队列', count: 5 }
      ]
    }
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
    background: rgba(0,0,0,0.4);
  }
  
  .content {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    background: #fff;
    border-radius: 20rpx 20rpx 0 0;
    max-height: 70vh;
    
    .header {
      padding: 30rpx;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 1px solid #f0f0f0;
      
      .title {
        font-size: 32rpx;
        color: #333;
        font-weight: bold;
      }
      
      .close {
        font-size: 40rpx;
        color: #999;
        padding: 0 20rpx;
      }
    }
    
    .queue-list {
      max-height: calc(70vh - 100rpx);
      padding: 20rpx;
      
      .queue-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 20rpx;
        
        .queue-item {
          background: #f5f7fa;
          padding: 30rpx 20rpx;
          border-radius: 12rpx;
          text-align: center;
          
          .queue-name {
            font-size: 32rpx;
            color: #333;
            font-weight: bold;
            margin-bottom: 10rpx;
            display: block;
          }
          
          .count {
            font-size: 26rpx;
            color: #666;
          }
          
          &:active {
            background: #e6f7ff;
          }
        }
      }
    }
  }
}
</style> 