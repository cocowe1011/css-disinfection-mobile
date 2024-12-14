<script>
	export default {
		onLaunch: function() {
			// 检查登录状态
			const token = uni.getStorageSync('token');
			if (!token) {
				uni.reLaunch({
					url: '/pages/login/login'
				});
			}
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style lang="scss">
@import '@/styles/variables.scss';

/*每个页面公共css */
page {
	background-color: #f5f5f5;
	font-family: $font-family;
}
</style>
