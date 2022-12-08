<template>
	<view class="page-projects-index">
		<view :prop="markerList" :change:prop="amap.updateEcharts" id="amap"></view>
	</view>

</template>

<script>
	const start = 'static/ITkoala-amap/start.png'

	export default {
		data() {
			return {
				markerList: [],
				dataIndex: ''
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
				this.markerList = [{
						lat: 30.590857,
						lng: 114.393329,
						icon: start
					},
					{
						lat: 30.53114,
						lng: 114.199008,
						icon: start
					},
					{
						lat: 30.620111,
						lng: 114.307842,
						icon: start
					}
				]
			},
			//地图点击回调事件
			onViewClick(params) {
				this.dataIndex = params.dataIndex
			}
		}
	}
</script>

<script module="amap" lang="renderjs">
	const selectedStart = 'static/ITkoala-amap/selectedStart.png' //选中的图片

	export default {
		data() {
			return {
				amapWebServiceKey: '55ad4700fed8a7d878cb5cebb83f093b',
				amapJsApiKey: '03ab0effeba13329fc44ce6559bc6a3d',
				map: null,
				ownerInstanceObj: null //service层对象
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
							if (prevMarker) {
								prevMarker.setIcon(prevIcon)
							}
							prevIcon = item.icon
							prevMarker = marker
							marker.setIcon(selectedStart)
							this.dataIndex = index
							this.onClick(this.ownerInstanceObj)
						})

						this.map.add(marker)
					}

				})
			},
			updateEcharts(newValue, oldValue, ownerInstance, instance) {
				// 监听 service 层数据变更
				this.ownerInstanceObj = ownerInstance
				this.initMarkers()
			},
			onClick(ownerInstance) {
				// 调用 service 层的方法
				ownerInstance.callMethod('onViewClick', {
					dataIndex: this.dataIndex
				})
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

		.infoWindow-wrap {
			margin-left: 50px;
			color: #f00;
		}
	}
</style>
