<template>
	<view class="block">
		<uni-nav-bar right-text="" title="区块"></uni-nav-bar>
		<view class="show_consensus fr">
			<view class="fl font12 consensus" style="line-height: 1.8rem;">隐藏共识奖励区块</view>
			<switch class="fl" style="transform:scale(0.5)" checked />
		</view>

		<view class="cb block_list">
			<uni-list v-for="item of blockList" style="border-top: 1px solid #c1c1c1;">
				<uni-list-item @click="toBlockInfo(item)" :title="'高度:'+item.height+' 时间:'+item.createTime" :note="'大小:'+item.size+' 奖励:'+item.reward+' 节点:'+item.nodeName"
				 :show-badge="item.txCount>1" :badge-text="item.txCount">
				</uni-list-item>
			</uni-list>
		</view>
		<uni-load-more :status="loadStatus"></uni-load-more>

	</view>
</template>

<script>
	import moment from 'moment'
	import uniNavBar from "@/components/uni-nav-bar/uni-nav-bar.vue"
	import uniDrawer from '@/components/uni-drawer/uni-drawer.vue'
	import uniList from "@/components/uni-list/uni-list.vue"
	import uniListItem from "@/components/uni-list-item/uni-list-item.vue"
	import uniLoadMore from "@/components/uni-load-more/uni-load-more.vue"
	import {
		timesDecimals,
		getLocalTime
	} from "@/static/api/util.js"
	export default {
		data() {
			return {
				hideSwitch: false, //隐藏单笔交易滑块
				blockList: [], //块列表数据
				loadStatus: "loading", //加载状态
				//分页信息
				pager: {
					total: 0,
					page: 1,
					rows: 15,
				}
			};
		},
		components: {
			uniNavBar,
			uniDrawer,
			uniList,
			uniListItem,
			uniLoadMore
		},
		onLoad() {
			this.getBlockList(this.pager.page, this.pager.rows, this.hideSwitch, '')
		},
		methods: {

			onPageScroll: function(Object) {
				//console.log(Object.scrollTop);//实时获取到滚动的值 
			},

			/**
			 * 获取块列表
			 */
			getBlockList(pager, rows, isShow, packAddress) {

				const url = 'https://public1.nuls.io';
				const params = {
					"jsonrpc": "2.0",
					"method": "getBlockHeaderList",
					"params": [1, pager, rows, isShow, packAddress],
					"id": Math.floor(Math.random() * 1000)
				};
				console.log(params);
				this.$http.post(url, params).then(res => {
					console.log(res);
					if (res.data.hasOwnProperty("result")) {
						for (let item of res.data.result.list) {
							item.reward = Number(timesDecimals(item.reward, 8)).toFixed(3);
							item.createTime = moment(getLocalTime(item.createTime * 1000)).format('YYYY/MM/DD HH:mm:ss');
							item.nodeName = item.agentAlias ? item.agentAlias : item.agentId.length < 10 ? item.agentId : '种子节点';

						}
						this.blockList = res.data.result.list;
						this.pager.total = res.data.result.totalCount;
					}
				})
			},

			/**
			 * 分页功能
			 **/
			pagesList() {
				this.blockLoading = true;
				this.getBlockList(this.pager.page, this.pager.rows, this.hideSwitch, '')
			},

			/**
			 * 隐藏单笔交易滑块
			 **/
			hideOneList() {
				this.blockLoading = true;
				this.getBlockList(this.pager.page, this.pager.rows, this.hideSwitch, '');
			},

			/**
			 * 详情跳转
			 * @param {Object} blockInfo
			 */
			toBlockInfo(blockInfo) {
				uni.navigateTo({
					url: 'blockInfo?height=' + blockInfo.height
				});
			},

			/**
			 * url 连接跳转
			 * @param name
			 * @param parmes
			 */
			toUrl(name, parmes) {
				let newQuery = {};
				console.log(name);
				if (name === 'consensusInfo') {
					newQuery = {
						hash: parmes
					};
					console.log(newQuery)
				} else {
					newQuery = {
						height: parmes
					}
				}
				this.$router.push({
					name: name,
					query: newQuery
				})

			}

		}
	};
</script>

<style lang="less">
	@import "./../../static/css/base.less";

	.content {
		text-align: center;
		height: 400upx;
		margin-top: 200upx;

		.show_consensus {
			.consensus {}
		}

		.block_list {
			.uni-list {
				border-top: 1px solid #5E6983;
			}
		}
	}
</style>
