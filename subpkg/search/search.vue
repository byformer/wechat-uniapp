<template>
	<view>
		<view class="search-box">
			<!-- 使用 uni-ui 提供的搜索组件 -->
			<uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
		</view>
		<!-- 搜索建议列表 -->
		<view class="sugg-list" v-if="searchResults.length !== 0">
			<view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				timer: null,
				kw: '',
				// 搜索的结果列表
				searchResults: [],
				// 搜索历史的数组
				historyList: []
			};
		},
		methods: {

			// 跳转到分包中的搜索页面
			input(e) {
				// 清除 timer 对应的延时器
				clearTimeout(this.timer)
				// 重新启动一个延时器，并把 timerId 赋值给 this.timer
				this.timer = setTimeout(() => {
					// 如果 500 毫秒内，没有触发新的输入事件，则为搜索关键词赋值
					this.kw = e
					this.getSearchList()
					console.log(this.kw)
				}, 500)
			},
			async getSearchList() {
				// 判断搜索关键词是否为空
				if (this.kw.length === 0) {
					this.searchResults = []
					return
				}

				const {
					data: res
				} = await uni.$http.get('/api/public/v1/goods/qsearch', {
					query: this.kw
				})
				if (res.meta.status !== 200) return uni.$showMsg()
				this.searchResults = res.message

				this.saveSearchHistory()
			},
			gotoDetail(item) {
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			},
			saveSearchHistory() {
				// this.historyList.push(this.kw)

				const set = new Set(this.historyList)
				set.delete(this.kw)
				set.add(this.kw)

				this.historyList = Array.from(set)

				// 对搜索历史数据，进行持久化的存储
				uni.setStorageSync('kw', JSON.stringify(this.historyList))
			},
		},

	}
</script>

<style lang="scss">
	.search-box {
		position: sticky;
		top: 0;
		z-index: 999;
	}
	 .sugg-list {
	    padding: 0 5px;
	
	    .sugg-item {
	      display: flex;
	      align-items: center;
	      justify-content: space-between;
	      font-size: 12px;
	      padding: 13px 0;
	      border-bottom: 1px solid #efefef;
	
	      .goods-name {
	        white-space: nowrap;
	        overflow: hidden;
	        text-overflow: ellipsis;
	        margin-right: 3px;
	      }
	    }
	  }
</style>
