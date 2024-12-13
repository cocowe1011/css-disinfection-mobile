<template>
  <button 
    class="scan-button" 
    :class="{ loading }"
    @tap="handleTap"
    :disabled="loading || disabled"
  >
    <text class="iconfont icon-scan" v-if="!loading"></text>
    <loading-spin v-else size="28"></loading-spin>
    <text class="btn-text">{{ loading ? '扫描中...' : '扫码添加' }}</text>
  </button>
</template>

<script>
import LoadingSpin from './loading-spin.vue'

export default {
  name: 'ScanButton',
  components: {
    LoadingSpin
  },
  props: {
    loading: Boolean,
    disabled: Boolean
  },
  methods: {
    handleTap() {
      this.$emit('scan')
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/styles/variables.scss';

.scan-button {
  display: flex;
  align-items: center;
  justify-content: center;
  background: $primary-gradient;
  color: #fff;
  font-size: 28rpx;
  padding: 20rpx 40rpx;
  border-radius: 40rpx;
  border: none;
  transition: opacity $transition-fast;
  
  .iconfont {
    font-size: 32rpx;
    margin-right: 12rpx;
  }
  
  &.loading {
    opacity: 0.8;
  }
  
  &:active {
    opacity: 0.9;
  }
  
  &[disabled] {
    opacity: 0.6;
    pointer-events: none;
  }
}
</style> 