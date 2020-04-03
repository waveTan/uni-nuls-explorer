<template>
	<view class="block_info">
		<uni-nav-bar @clickLeft="back" left-icon="back" left-text="返回" title="区块详情"></uni-nav-bar>
		<view class="height">
			 {{address}}
		</view>
		<view class="info">
			<view class="title font16">
				基本信息
			</view>
			<view class="info_list cb">
				<view class="fl font14 w">块hash</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.hashs}}
					<view class="fr font14 click ml_20">复制</view>
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">出块节点</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.agentAlias ? nodeInfo.agentAlias:nodeInfo.agentId}}
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">打包地址</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.packingAddresss}}
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">交易数量</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.txCount}}
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">大小</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.size}}
					<span> Bytes</span>
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">块奖励</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.reward}}
					<span class="fCN"> NULS</span>
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">轮次/编号</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.roundIndex}} / {{nodeInfo.packingIndexOfRound}}
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">手续费</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.totalFee}}
					<span class="fCN"> NULS</span>
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">时间</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.createTime}}
				</view>
			</view>
			<view class="info_list cb">
				<view class="fl font14">共识奖励</view>
				<view class="fr font14 ml_20">
					{{nodeInfo.totalReward}}
					<span class="fCN"> NULS</span>
				</view>
			</view>
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
		getLocalTime,
		superLong
	} from "@/static/api/util.js"
	export default {
		data() {
			return {
				address: this.$route.query.address, //高度
				nodeInfo: {}, //块信息
				loadStatus: "more", //加载状态
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
			console.log(this.address)
			//this.getHeaderByHeight(Number(this.address))
		},
		methods: {

			back() {
				console.log("back");
				window.history.back(-1); 
			},

			/**
			 * 复制方法
			 * @param sting
			 **/
			copy(sting) {
				copys(sting);
				this.$message({
					message: '复制成功',
					type: 'success',
					duration: 1000
				});
			},

			/**
			 * 根据高度获取块信息
			 */
			async getHeaderByHeight(height) {
				this.loadStatus = 'loading';
				const url = 'https://public1.nuls.io';
				const params = {
					"jsonrpc": "2.0",
					"method": "getHeaderByHeight",
					"params": [1, height],
					"id": Math.floor(Math.random() * 1000)
				};
				//console.log(params);
				this.$http.post(url, params).then(response => {
					console.log(response.data);
					if (response.data.hasOwnProperty("result")) {
						response.data.result.hashs = superLong(response.data.result.hash, 10);
						response.data.result.packingAddresss = superLong(response.data.result.packingAddress, 8);
						response.data.result.totalReward = timesDecimals(response.data.result.reward - response.data.result.totalFee,
							8);
						response.data.result.reward = timesDecimals(response.data.result.reward, 8);
						response.data.result.totalFee = timesDecimals(response.data.result.totalFee, 8);
						if (response.data.result.agentId.length > 10) {
							response.data.result.agentId = "种子节点"
						}

						response.data.result.createTime = moment(getLocalTime(response.data.result.createTime * 1000)).format(
							'YYYY-MM-DD HH:mm:ss');
						this.nodeInfo = response.data.result;
						this.loadStatus = 'noMore';
					}
				}).catch((error) => {
					console.log(error)
				})
			},

			/**
			 * 根据高度获取交易列表
			 */
			async getTxListByHeight(page, rows, height, type) {
				this.$post('/', 'getBlockTxList', [page, rows, height, type, ])
					.then((response) => {
						//console.log(response);
						if (response.hasOwnProperty("result")) {
							for (let item of response.result.list) {
								item.time = moment(getLocalTime(item.createTime * 1000)).format('YYYY-MM-DD HH:mm:ss');
								item.value = timesDecimals(item.value, 8);
								item.hashs = superLong(item.hash, 20);
								item.fees = timesDecimals(item.fee.value, 8);
							}
							this.txList = response.result.list;
							this.pager.total = response.result.totalCount;
						}
					}).catch((error) => {
						console.log(error)
					})
			},

			/**
			 * 分页功能
			 **/
			pagesList() {
				this.getTxListByHeight(this.pager.page, this.pager.rows, this.height, parseInt(this.typeRegion));
			},

			/**
			 * 获取交易类型
			 **/
			changeType(type) {
				this.typeRegion = type;
				this.getTxListByHeight(this.pager.page, this.pager.rows, this.height, parseInt(this.typeRegion));
			},

			/**
			 * url 连接跳转
			 * @param name
			 * @param parmes
			 */
			toUrl(name, parmes) {
				let newQuery = {};
				if (name === 'addressInfo') {
					newQuery = {
						address: parmes
					}
				} else if (name === 'rotationInfo') {
					newQuery = {
						rotation: parmes
					}
				} else {
					newQuery = {
						hash: parmes
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

	.block_info {
		.height {
			font-weight: 2rem;
			padding: 0.5rem 0.5rem;
		}

		.info {
			margin: 0 0.3rem 0 0.3rem;
			border: 1px solid #dfe4ef;

			.title {
				line-height: 2rem;
				text-align: left;
				background-color: #f5f6f9;
				height: 2rem;
				padding: 0 0 0 0.5rem;
				font-size: 0.8rem;
			}

			.info_list {
				border-bottom: 1px solid #dfe4ef;
				height: 1.8rem;
				line-height: 1.8rem;
				padding: 0 0.5rem;

				&:last-child {
					border-bottom: 0;
				}
			}
		}
	}
</style>
