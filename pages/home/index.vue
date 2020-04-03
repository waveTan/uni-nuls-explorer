<template>
	<view class="home">
		<uni-nav-bar>
			<view slot="left" class="iconfont icon-logo logo"></view>
			<view class="title">首页</view>
			<view slot="right" class="iconfont icon-big_set_icon set"></view>
		</uni-nav-bar>
		<view class="height">当前高度:{{heightInfo.networkHeight}}</view>
		<view class="search">
			<uni-search-bar :radius="10" @confirm="search" placeholder="地址/高度/txhash..."></uni-search-bar>
		</view>
		<view class="info">
			<view class="info-item">
				<view class="info-title">共识中节点</view>
				<view class="info-num">{{nodeInfo.consensusCount}}</view>
			</view>
			<view class="info-item">
				<view class="info-title">全网总委托</view>
				<view class="info-num">{{(nulsNumberInfo.consensusTotal/1000000000000).toFixed(2)}} W</view>
			</view>
			<view class="info-item">
				<view class="info-title">总发行量</view>
				<view class="info-num">{{(nulsNumberInfo.total/1000000000000).toFixed(2)}} W</view>
			</view>
			<view class="info-item">
				<view class="info-title">总流通量</view>
				<view class="info-num">{{(nulsNumberInfo.circulation/1000000000000).toFixed(2)}} W</view>
			</view>
		</view>
	</view>
</template>


<script>
	import uniNavBar from "@/components/uni-nav-bar/uni-nav-bar.vue"
	import uniSearchBar from '@/components/uni-search-bar/uni-search-bar.vue'
	export default {
		data() {
			return {
				heightInfo: {}, //高度信息
				nulsNumberInfo: {}, //nuls信息
				nodeInfo: {}, //节点信息
			};
		},
		components: {
			uniNavBar,
			uniSearchBar
		},
		onLoad() {
			this.getNewHeight();
			setInterval(() => {
				this.getNewHeight();
			}, 10000);
			this.getNodeNumber();
			this.getNULSNumber();
		},
		methods: {

			//获取最新高度
			getNewHeight() {
				const url = 'https://public1.nuls.io';
				const params = {
					"jsonrpc": "2.0",
					"method": "getInfo",
					"params": [1],
					"id": Math.floor(Math.random() * 1000)
				};
				this.$http.post(url, params).then(res => {
					//console.log(res)
					if (res.data.hasOwnProperty("result")) {
						this.heightInfo = res.data.result
					}
				}).catch(err => {
					console.log(err)
				})
			},

			/**
			 * 获取节点数量
			 */
			getNodeNumber() {
				const url = 'https://public1.nuls.io';
				const params = {
					"jsonrpc": "2.0",
					"method": "getConsensusNodeCount",
					"params": [1],
					"id": Math.floor(Math.random() * 1000)
				};
				this.$http.post(url, params)
					.then((response) => {
						//console.log(response);
						if (response.data.hasOwnProperty("result")) {
							this.nodeInfo = response.data.result;
							//this.$store.commit('SET_NODENUMBER', response.result);
						}
					})
			},

			/**
			 * 获取NULS数量信息
			 */
			getNULSNumber() {
				const url = 'https://public1.nuls.io';
				const params = {
					"jsonrpc": "2.0",
					"method": "getCoinInfo",
					"params": [1],
					"id": Math.floor(Math.random() * 1000)
				};
				this.$http.post(url, params)
					.then((response) => {
						console.log(response);
						if (response.data.hasOwnProperty("result")) {
							this.nulsNumberInfo = response.data.result
							//this.$store.commit('SET_NULSNUMBER', response.result);
						}
					})
			},
		}
	};
</script>

<style lang="less">
	@import url("./../../static/icon/iconfont.css");

	.home {
		text-align: center;

		.height {
			height: 2rem;
			font-size: 1rem;
			font-weight: bold;
			line-height: 2rem;
		}

		.info {

			.info-item {
				float: left;
				width: 25%;

				.info-title {
					font-size: 0.7rem;
					margin: 1rem 0 0.5rem 0;
				}

				.info-num {
					font-size: 0.8rem;
				}
			}
		}
	}

	.logo {
		color: #4CD964;
		font-size: 1.6rem;
	}

	.title {
		text-align: center;
		font-size: 0.8rem;
		width: 100%;
	}

	.set {
		font-size: 1.6rem;
	}
</style>
