<template>
	<view class="container">
		<map @tap="mapTap()" id="map" style="width: 100%; height: 100vh;" enable-overlooking=true enable-building :longitude="longitude" :latitude="latitude" :markers="markers" :polyline="polyline" :scale="scale" show-location>
			<cover-view slot="callout">
				<cover-view :marker-id="item.id" :key='index' v-for="(item,index) in markers">
					<cover-view class="position-relative" style="overflow: hidden;">
						<cover-image class="position-absolute" style="width:96rpx; height:140rpx;" :src='item.iconPath1'></cover-image>
					
						<cover-image :src="item.dogImg" alt="头像" style="top:8rpx; left: 10rpx;width: 78rpx; height: 78rpx; border-radius: 50%;position: absolute;">
						</cover-image>
						
					</cover-view>
				</cover-view>
			</cover-view>
			<cover-image src="/static/ding_right.png" class="d_r fedeIn" @click="handCover"></cover-image>

			<cover-view @click="modeDeviceid" style="right: 10px;
	bottom: calc(300px + env(safe-area-inset-bottom));position: absolute; width: 150px;height: 40px; background-color: #FF8C27;border-radius: 10px; text-align: center;line-height: 40px;color: #fff;">点我切换设备ID</cover-view>

			<cover-view @click="modeSwitch" style="right: 10px;
	bottom: calc(240px + env(safe-area-inset-bottom));position: absolute; width: 150px;height: 40px; background-color: #FF8C27;border-radius: 10px;color: #fff;text-align: center;line-height: 40px;">点我切换模式</cover-view>

			<cover-view class="mode_box">
				<cover-view v-if="showMode" class="mode-style" :class="{'hover' : index == idx}" v-for="(item,index) in modeData" :key="index" @click="handMode(index)">
		
						<cover-view class="title">{{item.title}}</cover-view>
						<cover-view class="sub">{{item.sub}}</cover-view>
			
				</cover-view>
			</cover-view>
		</map>
		<uni-popup ref="popup" background-color="#fff" :mask-click="false">
			<form @submit="onSubmit">
				<view style="display: flex; align-items: center;padding: 10px;">
					<text style="color: #FF8C27">请输入设备ID：</text>
					<input name="deviceid" style="width: 140px;height: 40px; border: 1px solid #FF8C27;" type="text"  :value="deviceid" />
				</view>
				<button form-type="submit" style="width: 100px;height: 40px; background-color: #FF8C27;border-radius: 10px; display: flex;justify-content: center;align-items: center;color: #fff;margin:10px auto 10px">确认</button>
			</form>
		</uni-popup>

		<view class="control-panel" v-show="controlShow">
			<view class="outer-layer">
				<view class="left">
					<view class="avatar-box">
						<image class="avatar" :src="avatar"></image>
					</view>
					<text class="nickname">当前定位：{{ positonType }}</text>
					<!-- <image class="dotted" src="/static/dotted.png"></image> -->
				</view>
				<view class="right">
					<image class="refresh" src="/static/refresh.png"></image>
					<text class="update-time">{{ diffMinutes }}</text>
					<image class="close fadeOut" src="/static/close.png" @click="closeHand"></image>
				</view>
			</view>
			<view class="inner-layer">
				<view class="left">
					<image class="location-icon" src="/static/location.png"></image>
					 <view class="txt-box">
						<view class="address">{{address}}\r\n</view>
						<view class="meter" v-if="meter">距离你约{{meter}}米</view>
					 </view>
				</view>
				<view class="right">
					<view class="action-button" @click="getRoute">
						<image class="button-icon" src="/static/route.png"></image>
						<text class="button-text">路线</text>
					</view>
					<view class="action-button mar" @click="navigate">
						<image class="button-icon" src="/static/navigate.png"></image>
						<text class="button-text">导航</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>

	export default {
		data() {
			return {
				controlShow: true,
				websocket: null,
				heartbeatInterval: null,
				latitude: null,
				equLongitude: null,
				equLatitude: null,
				longitude: null,
				scale: 16,
				markers: [],
				polyline: [],
				address: null,
				lastAddress: null,
				wsUrl: 'wss://www.d3inf.com',
				heartbeatInterval: null,
				heartbeatIntervalms: 30000,
				token: 'BjbzLHhhukAfCvN74ZEqp1CMHEaxtgi9chMz9zBjxcjfTmUEyJkJpuop8SBSATC4YMhw3jSMPu4xkbVRsXxN5BXQraL6C817aeqGfe29JCWjx1pNhPi3dc2yraHnd4dMk5DZhvYiz5hHi5MAq4WsQ3ay9xdHPRvF5v1Ybzavhuz3Z2ttPjdLZXQpXAzhxx9Y38BsC2WzRtgdZKfRmjvLerNmKJBqaB5E9Bwhp1GT4KAbZFw3pgFsF8pdpQ47dGPTQCCqPHBfdgExKrzRzyuMQfhDqXhw3iD4EoNLMQ1kmuJWcK3adEXdM6jqj9k8qLk8mPRdi4DMEuhZQcpqPkkv5NeBoRTf4VMD87N9RFBaEZGgVKh7EgBcmkGFzmaxCBpijcELv7mdByoZQMAuVsgXzSn7PHf9QrYqFSBei3SLhsoGE1LBBjuFy', 
				salt: '556cebc06941488096f8edace3c0b677',
				tu: 1, 
				socketStatus: false,
				devAddress: null,
				isRoute: false,
				avatar: '../../static/moren.jpg',
				deviceid:"14079706890",
				orgid: "openluat",
				meter: null,
				showMode:false,
				modeData: [
					{
						title: '休眠模式',
						sub: '关闭自动定时'
					},
					{
						title: '常规模式（比较耗电）',
						sub: '每3分钟自动定位一次'
					},
					{
						title: '智能模式',
						sub: '最快每30秒定位一次',
					}
				],
				idx:-1,
				inputValut: false,
				positonType: null,
				diffMinutes: 0,
				modeType: null
			};
		},
		onLoad() {
			this.getWebSocketConfig();
			this.getLocation();
			this.getLastAddress();
		},
		onReady() {
			this.map = uni.createMapContext("map", this); 
		},
		onUnload() {
			this.closeWebSocket();
		},
		methods: {
			getLocation() {
				uni.getLocation({
					type: 'gcj02',
					altitude: true,
					geocode: false,
					isHighAccuracy: true,
					success: res => {
						this.equLatitude = res.latitude
						this.equLongitude = res.longitude
						if (this.equLatitude && this.equLongitude) {
							this.drawLine();
						}
					},
					fail: err => {
						console.log('获取位置失败', err)
					}
				})
			},
			getWebSocketConfig() {
				uni.request({
					url: 'https://www.d3inf.com/iot/wsconfig',
					header: {
						'token': this.token,
						'salt': this.salt,
						'tu': this.tu
					},
					method: 'POST',
					success: (res) => {
						if (res.statusCode === 200 && res.data.code === 0) {
							const wsConfig = res.data.value;
							this.heartbeatIntervalms = wsConfig.timeout / 2;
							this.connectWebSocket(wsConfig);
						}
					},
					fail: (err) => {
						console.error('请求失败', err);
					},
				});
			},
			connectWebSocket(wsConfig) {
				this.websocket = uni.connectSocket({
					url: this.wsUrl + ':' + wsConfig.port + '/' + wsConfig.sub,
					protocol: [],
					header: {
						'token': this.token,
						'salt': this.salt,
						'tu': this.tu
					},
					success: () => {
						this.socketStatus = true;
					},
					fail: (err) => {
						this.socketStatus = false;
					},
				});

				this.websocket.onOpen(() => {
					this.socketStatus = true;
					this.startHeartbeat();
					this.requestDeviceLocation(true)
				});
				this.websocket.onClose(() => {
					this.socketStatus = false;
					this.stopHeartbeat();
				});

				this.websocket.onError((err) => {
					this.socketStatus = false;
					this.stopHeartbeat();
				});

				this.websocket.onMessage((message) => {
					this.socketStatus = true;
					this.handleWebSocketMessage(message.data);
				});
			},
			requestDeviceLocation(start) {
				const message = {
					"cmd": "location",
					"deviceid": this.deviceid,
					"start": start,
				};
				this.sendMessageToWebSocket(message);
			},
			sendMessageToWebSocket(message) {
				if (!this.socketStatus && this.websocket) {
					this.websocket.close({
						success: () => {
							console.log('WebSocket连接已关闭')
						},
						fail: (err) => {
							console.error('WebSocket关闭失败')
						},
					});
					this.getWebSocketConfig();
				}
				if (this.websocket) {
					this.websocket.send({
						data: JSON.stringify(message),
						success: () => {
							this.socketStatus = true;
							this.resetHeartbeat();
						},
						fail: (err) => {
							this.socketStatus = false;
						},
					});
				} else {
					this.socketStatus = false;
				}
			},
			startHeartbeat() {
				this.heartbeatInterval = setInterval(() => {
					this.sendMessageToWebSocket({
						type: 'heartbeat'
					});
				}, this.heartbeatIntervalms);
			},
			stopHeartbeat() {
				if (this.heartbeatInterval) {
					clearInterval(this.heartbeatInterval);
					this.heartbeatInterval = null;
				}
			},
			resetHeartbeat() {
				this.stopHeartbeat();
				this.startHeartbeat();
			},
			closeWebSocket() {
				if (this.websocket) {
					this.websocket.close();
				}
			},
			handleWebSocketMessage(data) {
				const message = JSON.parse(data);
				if (message.location) {
					const lat = message.location[message.location.length - 1].lat;
					const lng = message.location[message.location.length - 1].lng;
					if(lat != this.latitude || lng != this.longitude) {
						this.latitude = lat;
						this.longitude = lng;
						this.getLastAddress(message.location)
					}
				}
				this.resetHeartbeat();
			},
			drawLine() {
				this.polyline.length = 0; //清除之前绘制的线条
				if (this.isRoute && this.equLatitude != null && this.latitude != null) {
					this.polyline.push({
						'points': [{
								'latitude': this.latitude,
								'longitude': this.longitude
							},
							{
								'latitude': this.equLatitude,
								'longitude': this.equLongitude
							}
						],
						'color': '#bd000036',
						'arrowLine': true,
						'width': 4
					});
					this.markers = [{
						id: 1,
						latitude: this.latitude,
						longitude: this.longitude,
						width: 24,
						height: 35,
						iconPath: '/static/touming.png',
						iconPath1: '/static/avat_bg.png',
						dogImg: this.avatar,
						customCallout: {
							anchorX: -1,
							anchorY: 46,
							display: "ALWAYS"
						},
					},{
						id: 2,
						latitude: this.equLatitude,
						longitude: this.equLongitude,
					}];
					this.zoomMap();
				} else if (this.latitude != null) { 
					
					this.markers = [{
						id: 1,
						latitude: this.latitude,
						longitude: this.longitude,
						width: 24,
						height: 35,
						iconPath: '/static/touming.png',
						iconPath1: '/static/avat_bg.png',
						dogImg: this.avatar,
						customCallout: {
							anchorX: -1,
							anchorY: 46,
							display: "ALWAYS"
						},
					}];
				} else {
					this.markers = [{
						id: 1,
						latitude: this.equLatitude,
						longitude: this.equLongitude,
						width: 24,
						height: 35,
						iconPath: '/static/touming.png',
						iconPath1: '/static/avat_bg.png',
						dogImg: this.avatar,
						customCallout: {
							anchorX: -1,
							anchorY: 46,
							display: "ALWAYS"
						},
					}];
				}
			},
			getRoute() {
				this.isRoute = true;
				this.getLastAddress(true);
			},
			navigate() {
				this.isRoute = false;
				this.requestDeviceLocation(false);
				this.getLastAddress();
				if (this.latitude && this.longitude) {
					uni.openLocation({
						latitude: this.latitude,
						longitude: this.longitude,
						address: this.address,
					});
				}
			},
			getLastAddress(res) {
				uni.request({
					url: 'https://www.d3inf.com/jt/location/lastaddr',
					header: {
						'token': this.token,
						'salt': this.salt,
						'tu': this.tu
					},
					data: {
						'fmt': 1,
						"deviceid": this.deviceid,
						'orgid': this.orgid,
						lat: res && this.equLatitude || null,
						lng: res && this.equLongitude || null
					},
					method: 'POST',
					timeout: 5000,
					success: (res) => {
						if (res.statusCode === 200 && res.data.code === 0) {
							const {address, lat, lng, type, time, meter = 0} = res.data.value;
							this.address = address;
							this.latitude = lat;
							this.longitude = lng;
							this.meter = meter ? parseInt(meter) : null
							this.positonType = ['gps', 'wifi', 'lbs', '建栋'][type -1]
							// const timeDiff = (new Date().getTime() - new Date(time).getTime()) / 1000;
							// const timeDiffMinutes = Math.floor(timeDiff / 60);
							// if (timeDiffMinutes >= 59) {
							// 	this.diffMinutes =  Math.floor(timeDiffMinutes / 60) + '小时前'
							// } else {
							// 	this.diffMinutes =   timeDiffMinutes + '分钟前'
							// }
							this.diffMinutes = time

							this.drawLine();
						} else {
							uni.showToast({
								title: res.data.value,
								icon: "none",
								duration: 2000
							});
						}
					}
				});
			},
			getDistance(lat1, lng1, lat2, lng2) {
				const R = 6371; // 地球半径，单位为千米
				const φ1 = (lat1 * Math.PI) / 180;
				const φ2 = (lat2 * Math.PI) / 180;
				const Δφ = ((lat2 - lat1) * Math.PI) / 180;
				const Δλ = ((lng2 - lng1) * Math.PI) / 180;

				const a =
					Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
					Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
				const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
				return R * c;
			},
			zoomMap() {
				const distance = this.getDistance(
					this.latitude,
					this.longitude,
					this.equLatitude,
					this.equLongitude
				);
				const mapScale = this.calculateScaleLevel(distance);
				const centerLat = (this.latitude + this.equLatitude) / 2;
				const centerLng = (this.longitude + this.equLongitude) / 2;

				if (this.scale != mapScale) {
				    this.scale = mapScale;
				}
			},

			calculateScaleLevel(distance) {
				if (distance < 1) {
					return 20; // 距离小于 1 km，缩放级别设为 20
				} else if (distance < 5) {
					return 18;
				} else if (distance < 10) {
					return 16;
				} else if (distance < 50) {
					return 14;
				} else if (distance < 100) {
					return 12;
				} else if (distance < 200) {
					return 10;
				} else if (distance < 400) {
					return 8;
				} else if (distance < 600) {
					return 6;
				} else if (distance < 800){
					return 4;
				} else {
					return 3;
				}
			},
			closeHand() {
				// this.controlShow = false
			},
			handCover() {
				this.controlShow = true
				this.drawLine();//清除路线并将设置markers
				this.scale = 16;//调整缩放等级为默认值
				// this.requestDeviceLocation(false);//关闭设备定位上报，关闭后socket不会再收到设备定位信息，模拟数据关闭无效
				// this.map.moveToLocation();

				// this.getWebSocketConfig();
				// this.getLocation();
				// this.getLastAddress();
			},
			modeSwitch() {
				this.$refs.popup.close()
				this.showMode = !this.showMode;
			},
			handMode(idx) {
				this.idx = idx
				let handArr = [3, 1, 2]
				uni.request({
					url: 'https://www.d3inf.com/iot/cmd/send',
					header: {
						'token': this.token,
						'salt': this.salt,
						'tu': this.tu
					},
					data: {
						"deviceid": this.deviceid,
						'orgid': this.orgid,
						'cmd': handArr[idx]
					},
					method: 'POST',
					timeout: 5000,
					success: (res) => {
						if (res.statusCode === 200 && res.data.code === 0) {
							this.showMode = false;
						} else {
							uni.showToast({
								title: res.data.value,
								icon: 'none',
								duration: 2000
							});
						}
					},
					fail: (err) => {
						console.error('请求失败', err);
						uni.showToast({
							title: res.data.value,
								icon: 'none',
								duration: 2000
							});
					},
				});
			},
			mapTap() {
				this.showMode = false
			},
			onSubmit(e) {
				this.deviceid = e.detail.value.deviceid
				this.$refs.popup.close()
				this.getWebSocketConfig();
				this.getLocation();
				this.getLastAddress();
			},
			modeDeviceid() {
				this.showMode = false
				this.$refs.popup.open('center')
			}
		}
	};
</script>