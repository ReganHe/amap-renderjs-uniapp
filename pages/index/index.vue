<template>
	<view class="page-projects-index">
		<view :prop="markerList" :change:prop="amap.updateEcharts" id="amap"></view>
		<view id="search">
			<view class="query-key">
				<input />
			</view>
			<view class="query-other">
				<view class="city-status">
					<input />
					<input />
				</view>
				<view class="current-position" @click="handleLocation">
					<image class="image" src="@/static/ITkoala-amap/position.png" mode="widthFix"></image>
				</view>

			</view>
		</view>
		<view id="infoWindow" v-if="infoWindowVisible">
			<view class="infoWindow-header">
				<image class="close" src="@/static/ITkoala-amap/close.png" mode="widthFix"
					@click="infoWindowVisible=false"></image>
			</view>
			<view class="infoWindow-content">
				<view class="project-info">
					<view class="projectName">{{ currentItem.projectName}}</view>
					<view class="command" @click="handleDetails">查看详情</view>
				</view>
				<view class="project-tags">
					<view v-if="currentItem.area" class="project-tag area">{{currentItem.area}}</view>
					<view v-if="currentItem.alarmCount" class="project-tag alarm">报警</view>
					<view v-if="currentItem.status===2" class="project-tag expired">已过期</view>
				</view>
			</view>

		</view>
	</view>

</template>

<script>
	export default {
		data() {
			return {
				model: {
					queryKey: '',
					cityId: null,
					status: null
				},
				markerList: [],
				infoWindowVisible: true,
			}
		},
		mounted() {
			this.$nextTick(() => {
				this.getMapData()
			})
		},
		methods: {
			// 模拟从后台获取地图数据
			getMapData() {
				setTimeout(() => {
					this.markerList = [{
							lat: 30.590857,
							lng: 114.393329,
							projectName: 'project1',
							icon: 'static/ITkoala-amap/start.png'
						},
						{
							lat: 30.53114,
							lng: 114.199008,
							projectName: 'project2',
							icon: 'static/ITkoala-amap/start.png'
						},
						{
							lat: 30.620111,
							lng: 114.307842,
							projectName: 'project3',
							icon: 'static/ITkoala-amap/start.png'
						}
					];
				}, 1000);
			},
			handleLocation() {
				console.log('handleLocation')
			},
			handleDetails() {
				console.log('handleDetails', this.currentItem)
			}
		}
	}
</script>

<script module="amap" lang="renderjs">
	export default {
		data() {
			return {
				amapWebServiceKey: '55ad4700fed8a7d878cb5cebb83f093b',
				amapJsApiKey: '03ab0effeba13329fc44ce6559bc6a3d',
				map: null,
				ownerInstanceObj: null, //service层对象
				currentItem: {
					projectName: '科学岛高压管道迁改施工',
					alarmCount: 1,
					status: 2,
					statusDesc: '已过期',
					area: '江夏区'
				} //当前点击的对象
			}
		},
		mounted() {
			if (typeof window.AMap === 'function') {
				this.initAmap()
			} else {
				// 动态引入较大类库避免影响页面展示
				const script = document.createElement('script')
				script.src = `https://webapi.amap.com/maps?v=1.4.15&key=${this.amapWebServiceKey}`
				script.onload = this.initAmap.bind(this)
				document.head.appendChild(script)
			}
		},
		methods: {
			initAmap() {
				this.map = new AMap.Map('amap', {
					resizeEnable: true,
					center: [114.304569, 30.593354],
					zooms: [4, 18], //设置地图级别范围
					zoom: 11
				})

				this.initMarkers()
			},
			//初始化标记点
			initMarkers() {
				let prevMarker = null
				let prevIcon = null
				this.markerList.forEach((item, index) => {
					if (!!item.icon) {
						//添加点标记
						let marker = new AMap.Marker({
							position: new AMap.LngLat(item.lng, item.lat),
							offset: new AMap.Pixel(-13, -30),
							icon: item.icon
						})

						marker.on('click', (e) => {
							this.currentItem = item
							if (!!prevMarker) {
								prevMarker.setIcon(prevIcon)
							}
							prevIcon = item.icon
							prevMarker = marker
							marker.setIcon('static/ITkoala-amap/selectedStart.png')
							setTimeout(() => {
								this.infoWindowVisible = true
							}, 100)
						})

						this.map.add(marker)
					}

				})
			},
			updateEcharts(newValue, oldValue, ownerInstance, instance) {
				// 监听 service 层数据变更
				this.ownerInstanceObj = ownerInstance
				this.initMarkers()
			}
		}
	}
</script>

<style lang="scss" scoped>
	.page-projects-index {
		padding-bottom: env(safe-area-inset-bottom);
		height: calc(100vh - env(safe-area-inset-bottom));
		background-color: #F5F7FA;

		#amap {
			width: 100%;
			height: 100%;
		}

		#search {
			position: absolute;
			top: 16rpx;
			left: 28rpx;
			right: 28rpx;

			.query-key {
				height: 64rpx;
				line-height: 64rpx;
				background: #FFFFFF;
				box-shadow: 0px 4px 10px 0px rgba(0, 0, 0, 0.15);
				border-radius: 8rpx;
			}

			.query-other {
				margin-top: 16rpx;
				display: flex;
				justify-content: space-between;

				.city-status {
					height: 64rpx;
					line-height: 64rpx;
					background: #FFFFFF;
					box-shadow: 0px 4px 10px 0px rgba(0, 0, 0, 0.15);
					border-radius: 8rpx;
					display: flex;
				}

				.current-position {
					width: 64rpx;
					height: 64rpx;
					background: #FFFFFF;
					box-shadow: 0px 4px 10px 0px rgba(0, 0, 0, 0.15);
					border-radius: 8rpx;
					display: flex;
					justify-content: center;
					align-items: center;

					.image {
						width: 30rpx;
						height: 36rpx;
					}

				}
			}
		}

		#infoWindow {
			position: absolute;
			bottom: 16rpx;
			left: 28rpx;
			right: 28rpx;
			padding: 32rpx 16rpx 24rpx 28rpx;
			background: #fff;
			box-shadow: 0px 4px 10px 0px rgba(0, 0, 0, 0.08);
			border-radius: 16rpx;



			.infoWindow-header {
				text-align: right;

				.close {
					margin-right: 24rpx;
					width: 32rpx;
					height: 32rpx;
				}
			}

			.infoWindow-content {
				margin-top: 32rpx;

				.project-info {
					display: flex;
					justify-content: space-between;
					align-items: center;

					.projectName {
						font-family: PingFangSC-Medium;
						font-size: 32rpx;
						font-weight: normal;
						line-height: 40rpx;
						letter-spacing: 0px;
						color: #3D3D3D;

					}

					.command {
						height: 60rpx;
						line-height: 60rpx;
						width: 160rpx;
						text-align: center;
						background: #0052D9;
						border-radius: 16rpx;
						font-family: PingFangSC-Regular;
						font-size: 28rpx;
						font-weight: normal;
						text-align: center;
						letter-spacing: 0px;
						color: #FFFFFF;
					}
				}

				.project-tags {
					margin-top: 16rpx;
					display: flex;

					.project-tag {
						margin-right: 8rpx;
						padding: 4rpx 16rpx;
						border-radius: 8px;
						color: #FFFFFF;
						font-family: PingFangSC-Medium;
						font-size: 12px;
						font-weight: normal;
						letter-spacing: 0px;

						&.area {
							background: #00B578;
						}

						&.alarm {
							background: #FA5151;
						}

						&.expired {
							background: #FF8F1F;
						}
					}
				}
			}
		}
	}
</style>
