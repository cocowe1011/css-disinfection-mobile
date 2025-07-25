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
      :refresher-enabled="true"
      :refresher-triggered="isRefreshing"
      @refresherrefresh="onRefresh"
      :style="{ opacity: pageReady ? 1 : 0 }"
    >
      <view class="content">
        <!-- 订单信息卡片 -->
        <order-card :order="currentOrder"></order-card>
        
        <!-- 暂无数据提示 -->
        <view v-if="!currentOrder.id" class="empty-state">
          <text class="iconfont icon-empty"></text>
          <text class="empty-text">暂无队列数据</text>
        </view>
        
        <!-- 队列区域列表 -->
        <view v-else class="queue-areas">
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
    
    <!-- 报警日志悬浮按钮 -->
    <view class="fab-btn alarm-fab" :class="{'has-unread-alarms': unreadAlarmCount > 0}" @tap="toggleAlarmModal">
      <text class="fab-text">报警</text>
      <text class="fab-text">日志</text>
      <view v-if="unreadAlarmCount > 0" class="alarm-badge">{{ unreadAlarmCount }}</view>
    </view>
    
    <!-- 报警日志弹窗 -->
    <view class="modal-overlay" v-if="showAlarmModal" @tap="toggleAlarmModal">
      <view class="alarm-modal-content" @tap.stop>
        <view class="alarm-modal-header">
          <text class="alarm-modal-title">消毒车间报警日志</text>
          <view class="alarm-actions">
            <view class="connection-status" :class="{'connected': wsStatus.isConnected, 'disconnected': !wsStatus.isConnected}">
              <text class="status-dot"></text>
              <text class="status-text">{{ wsStatus.isConnected ? '已连接' : '未连接' }}</text>
            </view>
            <view class="alarm-refresh-btn" @tap="refreshAlarmLogs">
              <text class="refresh-text">刷新</text>
            </view>
          </view>
        </view>
        
        <view class="alarm-modal-body">
          <scroll-view scroll-y="true" class="alarm-list">
            <!-- 空状态提示 -->
            <view class="empty-state alarm-empty-state" v-if="alarmLogs.length === 0">
              <text class="empty-icon">🚨</text>
              <text class="empty-text">暂无报警日志</text>
              <text class="empty-desc">电脑端产生报警时会自动推送到此处</text>
            </view>

            <!-- 报警日志列表 -->
            <view v-if="alarmLogs.length > 0">
              <view 
                v-for="alarm in alarmLogs" 
                :key="alarm.id" 
                class="alarm-card"
                :class="{'unread': alarm.unread}"
                @tap="markAlarmAsRead(alarm)"
              >
                <view class="alarm-card-header">
                  <view class="alarm-source">{{ alarm.source }}</view>
                  <view class="alarm-time">{{ formatAlarmTime(alarm.timestamp) }}</view>
                </view>
                <view class="alarm-message">{{ alarm.message }}</view>
                <view v-if="alarm.unread" class="unread-indicator">
                  <text class="unread-text">未读</text>
                </view>
              </view>
            </view>
          </scroll-view>
        </view>
        
        <view class="alarm-modal-footer">
          <view class="alarm-footer-actions">
            <view class="alarm-btn clear-btn" @tap="clearAllAlarms" v-if="alarmLogs.length > 0">
              清空日志
            </view>
            <view class="alarm-btn mark-read-btn" @tap="markAllAlarmsAsRead" v-if="unreadAlarmCount > 0">
              全部已读
            </view>
            <view class="alarm-btn close-btn" @tap="toggleAlarmModal">
              关闭
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import OrderCard from '@/components/order-card.vue'
import request from '@/config/request'
import AlarmWebSocketClient from '@/utils/WebSocketClient.js'

export default {
  components: {
    OrderCard
  },
  data() {
    return {
      isRefreshing: false,
      username: '张工',
      currentOrder: {},
      queueGroups: [],
      statusBarHeight: 0,
      pageReady: false,
      // WebSocket相关数据
      wsClient: null,
      wsStatus: {
        isConnected: false
      },
      alarmLogs: [],
      showAlarmModal: false
    }
  },
  computed: {
    // 未读报警数量
    unreadAlarmCount() {
      return this.alarmLogs.filter(alarm => alarm.unread).length;
    }
  },
  async onLoad() {
    // 获取状态栏高度
    const systemInfo = uni.getSystemInfoSync()
    this.statusBarHeight = systemInfo.statusBarHeight
    
    // 加载初始数据，不显示提示
    const hasOrder = await this.fetchOrderData(false)
    if (hasOrder) {
      await this.getQueueData()
    } else {
      this.queueGroups = []
    }
    
    // 初始化WebSocket连接
    this.initWebSocket()
    
    // 显示页面
    this.pageReady = true
  },
  beforeDestroy() {
    // 组件销毁前断开WebSocket连接
    if (this.wsClient) {
      this.wsClient.disconnect();
      this.wsClient = null;
    }
  },
  methods: {
    async getQueueData() {
      try {
        const res = await request.post('/queue_info/queryQueueList')
        if (res.code === '200' && Array.isArray(res.data)) {
          this.processQueueData(res.data)
        }
      } catch (error) {
        console.error('获取队列数据失败:', error)
      }
    },

    processQueueData(data) {
      // 定义区域分组
      const groups = {
        '上下货区域': ['上货区', '下货区-不解析', '下货区-立体库', '下货区-解析库', '小车一区', '小车二区', '小车三区'],
        '预热房区域1': ['A1', 'B1', 'C1', 'D1', 'E1', 'F1', 'G1'],
        '预热房区域2': ['A2', 'B2', 'C2', 'D2', 'E2', 'F2', 'G2'],
        '灭菌区': ['A3', 'B3', 'C3', 'D3', 'E3', 'F3', 'G3']
      }

      // 初始化结果数组
      const result = []
      let globalIndex = 1

      // 处理每个分组
      for (const [groupTitle, areaNames] of Object.entries(groups)) {
        const areas = []
        
        // 查找每个区域名称对应的数据
        for (const areaName of areaNames) {
          const areaData = data.find(item => item.queueName === areaName)
          if (areaData) {
            // 解析托盘信息
            let trayInfo = []
            try {
              trayInfo = areaData.trayInfo ? JSON.parse(areaData.trayInfo) : []
            } catch (e) {
              console.error('解析托盘信息失败:', e)
            }

            areas.push({
              id: globalIndex++,
              name: areaData.queueName,
              palletCount: trayInfo.length,
              queueId: areaData.id  // 使用接口返回的原始id
            })
          }
        }

        // 只添加有区域的分组
        if (areas.length > 0) {
          result.push({
            title: groupTitle,
            areas: areas
          })
        }
      }

      this.queueGroups = result
    },
    handleLogout() {
      uni.clearStorageSync()
      uni.reLaunch({
        url: '/pages/login/login'
      })
    },
    
    // 获取订单数据
    async fetchOrderData(showToast = true) {
      try {
        const res = await request.post('/order_info/getNowRunningOrder')
        if (res.code === '200') {
          if (res.data) {
            this.currentOrder = { ...res.data }
            if (showToast) {
              uni.showToast({
                title: '刷新成功',
                icon: 'success'
              })
            }
            return true
          } else {
            this.currentOrder = {}
            if (showToast) {
              uni.showToast({
                title: '暂无运行订单数据',
                icon: 'none'
              })
            }
            return false
          }
        } else {
          if (showToast) {
            uni.showToast({
              title: res.message || '获取数据失败',
              icon: 'none'
            })
          }
          return false
        }
      } catch (error) {
        console.error('获取运行订单数据失败:', error)
        if (showToast) {
          uni.showToast({
            title: '网络异常',
            icon: 'error'
          })
        }
        return false
      }
    },
    
    // 下拉刷新
    async onRefresh() {
      this.isRefreshing = true
      const hasOrder = await this.fetchOrderData()
      if (hasOrder) {
        await this.getQueueData()
      } else {
        this.queueGroups = []
      }
      this.isRefreshing = false
    },
    
    navigateToQueue(area) {
      uni.showLoading({
        title: '加载中...',
        mask: true
      })
      
      uni.navigateTo({
        url: `/pages/queue/queue?queueId=${area.queueId}&name=${area.name}`,
        success: () => {
          uni.hideLoading()
        },
        fail: () => {
          uni.hideLoading()
        }
      })
    },

    // ============ WebSocket和报警日志相关方法 ============
    // 初始化WebSocket连接
    initWebSocket() {
      this.wsClient = new AlarmWebSocketClient({
        onConnected: this.onWebSocketConnected,
        onDisconnected: this.onWebSocketDisconnected,
        onAlarmReceived: this.onAlarmReceived,
        onError: this.onWebSocketError
      });
      
      this.wsClient.connect();
    },

    // WebSocket连接成功
    onWebSocketConnected() {
      console.log('WebSocket连接成功');
      this.wsStatus.isConnected = true;
      uni.showToast({
        title: '连接服务器成功',
        icon: 'success',
        duration: 2000
      });
    },

    // WebSocket连接断开
    onWebSocketDisconnected() {
      console.log('WebSocket连接断开');
      this.wsStatus.isConnected = false;
      uni.showToast({
        title: '服务器连接断开',
        icon: 'none',
        duration: 2000
      });
    },

    // 收到报警消息
    onAlarmReceived(alarmLog) {
      // 添加到本地报警列表
      this.alarmLogs.unshift(alarmLog);
      // 保持日志数量在合理范围内
      if (this.alarmLogs.length > 100) {
        this.alarmLogs.pop();
      }
      // 显示通知
      uni.showToast({
        title: `报警: ${alarmLog.message}`,
        icon: 'error',
        position: 'top'
      });
      // 震动提醒（需要用户交互后才能生效）
      this.tryVibrate();
    },

    // WebSocket错误
    onWebSocketError(error) {
      console.error('WebSocket错误:', error);
      this.wsStatus.isConnected = false;
    },

    // 切换报警日志弹窗
    toggleAlarmModal() {
      this.showAlarmModal = !this.showAlarmModal;
    },

    // 刷新报警日志
    refreshAlarmLogs() {
      if (this.wsClient) {
        const status = this.wsClient.getConnectionStatus();
        this.wsStatus.isConnected = status.isConnected;
        this.alarmLogs = this.wsClient.getAlarmLogs();
      }
      
      uni.showToast({
        title: '刷新完成',
        icon: 'success',
        duration: 1000
      });
    },

    // 标记单个报警为已读
    markAlarmAsRead(alarm) {
      alarm.unread = false;
      
      // 同步到WebSocket客户端
      if (this.wsClient) {
        const clientAlarms = this.wsClient.getAlarmLogs();
        const clientAlarm = clientAlarms.find(a => a.id === alarm.id);
        if (clientAlarm) {
          clientAlarm.unread = false;
        }
      }
    },

    // 标记所有报警为已读
    markAllAlarmsAsRead() {
      this.alarmLogs.forEach(alarm => {
        alarm.unread = false;
      });
      
      // 同步到WebSocket客户端
      if (this.wsClient) {
        this.wsClient.markAlarmsAsRead();
      }
      
      uni.showToast({
        title: '已全部标记为已读',
        icon: 'success',
        duration: 1000
      });
    },

    // 清空所有报警日志
    clearAllAlarms() {
      uni.showModal({
        title: '确认清空',
        content: '确定要清空所有报警日志吗？',
        success: (res) => {
          if (res.confirm) {
            this.alarmLogs = [];
            
            // 同步到WebSocket客户端
            if (this.wsClient) {
              this.wsClient.clearAlarmLogs();
            }
            
            uni.showToast({
              title: '已清空报警日志',
              icon: 'success',
              duration: 1000
            });
          }
        }
      });
    },

    // 格式化报警时间
    formatAlarmTime(timestamp) {
      if (!timestamp) return '--';
      const date = new Date(timestamp);
      
      // 手动格式化时分秒，避免时区信息显示
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      const seconds = date.getSeconds().toString().padStart(2, '0');
      
      return `${hours}:${minutes}:${seconds}`;
    },

    // 尝试震动提醒（处理浏览器用户激活策略）
    tryVibrate() {
      try {
        // 检查是否支持震动
        if (typeof uni.vibrateLong === 'function') {
          uni.vibrateLong();
        }
      } catch (error) {
        // 静默处理震动失败（通常是因为缺少用户交互）
        console.log('震动提醒被浏览器阻止，需要用户交互后才能生效');
      }
    }
  },

  // 组件销毁前断开WebSocket连接
  beforeDestroy() {
    if (this.wsClient) {
      this.wsClient.disconnect();
      this.wsClient = null;
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
  
  .empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 100rpx 0;
    
    .iconfont {
      font-size: 120rpx;
      color: #ccc;
      margin-bottom: 20rpx;
    }
    
    .empty-text {
      font-size: 32rpx;
      color: #999;
    }
  }
}

/* 通用模态框覆盖层 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

/* 悬浮按钮基础样式 */
.fab-btn {
  position: fixed;
  right: 30rpx;
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4rpx 16rpx rgba(0, 0, 0, 0.2);
  z-index: 999;
  font-size: 24rpx;
  font-weight: 600;
  transition: all 0.3s ease;

  &:active {
    transform: scale(0.95);
    opacity: 0.8;
  }

  .fab-text {
    line-height: 1;
    font-size: 24rpx;
    font-weight: 600;
    color: #fff;
  }
}

/* 报警悬浮按钮 */
.alarm-fab {
  bottom: 140rpx;
  background-color: #f59e0b;

  &:active {
    background-color: #d97706;
  }
  
  // 脉冲效果 - 只在有未读报警时显示
  &.has-unread-alarms::before {
    content: '';
    position: absolute;
    top: -6rpx;
    left: -6rpx;
    right: -6rpx;
    bottom: -6rpx;
    background-color: #ef4444;
    border-radius: 50%;
    opacity: 0;
    animation: alarm-pulse 2s infinite;
    z-index: -1;
  }
  
  .alarm-badge {
    position: absolute;
    top: -10rpx;
    right: -10rpx;
    min-width: 36rpx;
    height: 36rpx;
    background-color: #ef4444;
    color: #fff;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20rpx;
    font-weight: 600;
    border: 2rpx solid #fff;
    box-shadow: 0 2rpx 8rpx rgba(239, 68, 68, 0.4);
  }
}

// 脉冲动画
@keyframes alarm-pulse {
  0% {
    opacity: 0;
    transform: scale(1);
  }
  50% {
    opacity: 0.3;
    transform: scale(1.2);
  }
  100% {
    opacity: 0;
    transform: scale(1.4);
  }
}

// 报警日志弹窗样式
.alarm-modal-content {
  width: 90%;
  max-width: 700rpx;
  height: 80vh;
  background: #ffffff;
  border-radius: 16rpx;
  overflow: hidden;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  color: #333;
}

.alarm-modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 24rpx;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  flex-shrink: 0;

  .alarm-modal-title {
    font-size: 32rpx;
    font-weight: 600;
    color: #1f2937;
  }

  .alarm-actions {
    display: flex;
    align-items: center;
  }

  .connection-status {
    display: flex;
    align-items: center;
    gap: 8rpx;
    padding: 6rpx 12rpx;
    border-radius: 12rpx;
    background: #f3f4f6;
    font-size: 22rpx;
    margin-right: 16rpx;

    .status-dot {
      width: 12rpx;
      height: 12rpx;
      border-radius: 50%;
      display: block;
    }

    &.connected .status-dot {
      background: #10b981;
    }

    &.disconnected .status-dot {
      background: #ef4444;
    }

    .status-text {
      color: #6b7280;
      font-weight: 500;
    }
  }

  .alarm-refresh-btn {
    background: #2563eb;
    color: #fff;
    padding: 8rpx 16rpx;
    border-radius: 8rpx;
    font-size: 22rpx;
    font-weight: 500;

    &:active {
      background: #1d4ed8;
    }
  }
}

.alarm-modal-body {
  flex: 1;
  overflow: hidden;
  background: #f8f9fa;
  display: flex;
  flex-direction: column;
}

.alarm-list {
  flex: 1;
  padding: 16rpx;
  box-sizing: border-box;
  overflow-x: hidden;
  overflow-y: auto;
}

.alarm-empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 100rpx 40rpx;
  text-align: center;

  .empty-icon {
    font-size: 100rpx;
    margin-bottom: 30rpx;
  }

  .empty-text {
    font-size: 32rpx;
    color: #666;
    margin-bottom: 20rpx;
  }

  .empty-desc {
    font-size: 28rpx;
    color: #999;
  }
}

.alarm-card {
  background: #ffffff;
  border-radius: 12rpx;
  padding: 24rpx;
  margin-bottom: 16rpx;
  box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.08);
  border-left: 4rpx solid #e5e7eb;
  transition: all 0.2s ease;

  &.unread {
    border-left-color: #ef4444;
    background: #fef2f2;
  }

  .alarm-card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12rpx;

    .alarm-source {
      font-size: 24rpx;
      color: #6366f1;
      font-weight: 600;
    }

    .alarm-time {
      font-size: 22rpx;
      color: #9ca3af;
    }
  }

  .alarm-message {
    font-size: 26rpx;
    color: #1f2937;
    line-height: 1.5;
    font-weight: 500;
    margin-bottom: 8rpx;
    word-wrap: break-word;
    word-break: break-all;
    white-space: normal;
    overflow-wrap: break-word;
  }

  .unread-indicator {
    text-align: right;

    .unread-text {
      font-size: 20rpx;
      color: #ef4444;
      background: #fee2e2;
      padding: 2rpx 8rpx;
      border-radius: 8rpx;
      font-weight: 500;
    }
  }
}

.alarm-modal-footer {
  padding: 20rpx 24rpx;
  background: #ffffff;
  border-top: 1px solid #e5e7eb;
  flex-shrink: 0;

  .alarm-footer-actions {
    display: flex;

    .alarm-btn {
      flex: 1;
      text-align: center;
      padding: 16rpx 0;
      font-size: 26rpx;
      font-weight: 500;
      border-radius: 8rpx;
      color: #fff;
      margin-right: 12rpx;
      
      &:last-child {
        margin-right: 0;
      }

      &.clear-btn {
        background: #ef4444;

        &:active {
          background: #dc2626;
        }
      }

      &.mark-read-btn {
        background: #10b981;

        &:active {
          background: #059669;
        }
      }

      &.close-btn {
        background: #6b7280;

        &:active {
          background: #4b5563;
        }
      }
    }
  }
}
</style> 