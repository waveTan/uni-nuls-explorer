<template>
	<view class="account">
		<uni-list v-for="item of addressList">
			<uni-list-item 
			@click="toAccountInfo(item)"
			:title="item.address" 
			:note="'总计:'+item.totalBalance+'总收入:'+item.totalIn+' 总支出:'+item.totalOut">
			
			</uni-list-item>
		</uni-list>
	</view>
</template>

<script>
	import uniList from "@/components/uni-list/uni-list.vue"
	import uniListItem from "@/components/uni-list-item/uni-list-item.vue"
	import {
		timesDecimals,
		getLocalTime
	} from "@/static/api/util.js"
	export default {
		data() {
			return {
				addressList: [],
				pager: {
					total: 0,
					page: 1,
					rows: 15,
				}
			};
		},
		components: {
			uniList,
			uniListItem
		},
		onLoad() {
			this.getAddressList(this.pager.page, this.pager.rows);
		},
		methods: {

			/**
			 * @disc: 获地址列表
			 * @params: page, rows
			 * @date: 2019-09-09 17:32
			 * @author: Wave
			 */
			getAddressList(pager, rows) {
				const url = 'https://public1.nuls.io';
				const params = {
					"jsonrpc": "2.0",
					"method": "getCoinRanking",
					"params": [1, pager, rows],
					"id": Math.floor(Math.random() * 1000)
				};
				//console.log(params);
				this.$http.post(url, params).then(response => {
					//console.log(response.data);
					if (response.data.hasOwnProperty("result")) {
						for (let item of response.data.result.list) {
							item.totalBalance = Number(timesDecimals(item.totalBalance, 8)).toFixed(3);
							item.balance = Number(timesDecimals(item.balance, 8)).toFixed(3);
							item.totalLock = Number(timesDecimals(item.consensusLock + item.timeLock, 8)).toFixed(3);
							item.totalOut = Number(timesDecimals(item.totalOut, 8)).toFixed(3);
							item.totalIn = Number(timesDecimals(item.totalIn, 8)).toFixed(3);
						}
						this.addressList = response.data.result.list;
						//console.log(this.addressList);
					}
				})
			},
			
			/**
			 * 详情跳转
			 * @param {Object} accountInfo
			 */
			toAccountInfo(accountInfo) {
				console.log(accountInfo)
				 uni.navigateTo({
					url: 'accountInfo?address=' + accountInfo.address
				}); 
			},
		}
	};
</script>

<style lang="less">
	.content {
		text-align: center;
		height: 400upx;
		margin-top: 200upx;

		.newslist ul li p {
			font-size: 14px;
			color: #555;
			line-height: 25px;
			height: 50px;
			overflow: hidden;
			transition: height .3s;
		}

	}
</style>
