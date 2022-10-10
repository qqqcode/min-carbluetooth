<template>
	<view class="content">
		<view class="content-main">
			<view class="bluetooth" @click="initBlue">
				<image :src="blueConImg" class="bluetooth-con" mode=""></image>
				<!-- <view class="bluetooth-con">
					<image class="bluetooth-icon" src="@/static/images/设置icon-暗.png" mode=""></image>
					<view class="bluetooth-p">
						蓝牙未连接
					</view>
				</view> -->
			</view>
			<view class="main-content">
				<view class="circle">
					<view class="circle-left ab"></view>
					<view class="circle-right ab"></view>
				</view>
			</view>
			<view class="text-area">
				<text class="title">{{title}}</text>
			</view>
			<view :class="{in:dialogshow}" class="dialog" @click="hideDialog">
				<view class="modal-dialog">
					<view class="dialog-body" @click.stop>
						<div class="dialog-header">
							蓝牙列表
						</div>
						<view class="dialog-content">
							<view v-if="blueDeviceList.length == 0" style="padding: 10px;">
								没有找到可连接蓝牙
							</view>
							<view class="blue-box">
								<view class="blue-list">
									<view class="blueinfo" v-for="(item, index) in blueDeviceList" :key="index"
										@click="connectBlue(item)">
										id:{{item.deviceId}}
										name:{{item.name}}
										localName:{{item.localName}}
									</view>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
	import {
		ref,
		onMounted,
		watch
	} from 'vue';
	
	const title = ref("Hello World");
	const blueStatus = ref(-1);
	const blueDeviceList = ref([])
	const dialogshow = ref(false);

	const deviceId = ref('')
	const blueConImgList = [
		"../../static/images/bluetoothNotCon.png",
		"../../static/images/bluetoothCon.png"
	]
	const blueConImg = ref()

	//连接蓝牙
	const connectBlue = (item) => {
		console.log(item)
		deviceId.value = item.deviceId
		uni.createBLEConnection({
			deviceId: deviceId.value,
			success(res) {
				console.log('连接成功')
				console.log(res)
				// 停止搜索
				stopDiscovery()
				blueStatus.value = 1;
				getServices()
			},
			fail(err) {
				console.log('连接失败')
				console.error(err)
			}
		})
	}

	//停止搜索附近蓝牙设备
	const stopDiscovery = () => {
		uni.stopBluetoothDevicesDiscovery({
			success(res) {
				console.log('停止成功')
				console.log(res)
			},
			fail(err) {
				console.log('停止失败')
				console.error(err)
			}
		})
	}

	//搜索到附近蓝牙设备回调函数
	const found = (res) => {
		console.log(res)
		blueDeviceList.value.push(res.devices[0])
		dialogshow.value = true;
	}

	//搜索附近蓝牙设备
	const discovery = () => {
		blueStatus.value = 0;
		uni.startBluetoothDevicesDiscovery({
			success(res) {
				console.log('开始搜索')
				// 开启监听回调
				uni.onBluetoothDeviceFound(found)
			},
			fail(err) {
				console.log('搜索失败')
				console.error(err)
			}
		})
	}

	//初始化蓝牙，打开蓝牙
	const initBlue = () => {
		blueConImg.value = blueConImg.value==blueConImgList[1]?blueConImgList[0]:blueConImgList[1];
		uni.openBluetoothAdapter({
			success(res) {
				console.log('初始化蓝牙成功')
				console.log(res)
				discovery();
			},
			fail(err) {
				console.log('初始化蓝牙失败')
				console.error(err)
			}
		})
	}

	//通过设备id获取服务信息
	const getServices = () => {
		uni.getBLEDeviceServices({
			deviceId: deviceId.value, // 设备ID
			success(res) {
				console.log(res)
			},
			fail(err) {
				console.error(err)
			}
		})
	}

	//通过设备id，服务id获取特征
	const getCharacteristics = () => {
		uni.getBLEDeviceCharacteristics({
			deviceId: deviceId.value, // 设备ID
			serviceId: '0000FFE0-0000-1000-8000-00805F9B34FB', // 服务UUID
			success(res) {
				console.log(res)
			},
			fail(err) {
				console.error(err)
			}
		})
	}

	//开启消息监听
	const notify = () => {
		uni.notifyBLECharacteristicValueChange({
			deviceId: deviceId.value,
			serviceId: '0000FFE0-0000-1000-8000-00805F9B34FB', // 服务UUID
			characteristicId: '0000FFE1-0000-1000-8000-00805F9B34FB', // 特征值
			success(res) {
				console.log(res)
				// 接受消息的方法
				listenValueChange()
			},
			fail(err) {
				console.error(err)
			}
		})
	}

	//ArrayBuffer转16进度字符串
	const ab2hex = (buffer) => {
		const hexArr = Array.prototype.map.call(
			new Uint8Array(buffer),
			function(bit) {
				return ('00' + bit.toString(16)).slice(-2)
			}
		)
		return hexArr.join('')
	}

	//将16进制的内容转成字符串
	const hexCharCodeToStr = (hexCharCodeStr) => {
		var trimedStr = hexCharCodeStr.trim();
		var rawStr = trimedStr.substr(0, 2).toLowerCase() === "0x" ? trimedStr.substr(2) : trimedStr;
		var len = rawStr.length;
		if (len % 2 !== 0) {
			alert("存在非法字符!");
			return "";
		}
		var curCharCode;
		var resultStr = [];
		for (var i = 0; i < len; i = i + 2) {
			curCharCode = parseInt(rawStr.substr(i, 2), 16);
			resultStr.push(String.fromCharCode(curCharCode));
		}
		return resultStr.join("");
	}

	//监听消息变化
	const listenValueChange = () => {
		uni.onBLECharacteristicValueChange(res => {
			// 结果
			console.log(res)
			// 结果里有个value值，该值为 ArrayBuffer 类型，所以在控制台无法用肉眼观察到，必须将该值转换为16进制
			let resHex = ab2hex(res.value)
			console.log(resHex)
			// 最后将16进制转换为ascii码，就能看到对应的结果
			let result = hexCharCodeToStr(resHex)
			console.log(result)
		})
	}

	const send = () => {
		// 向蓝牙设备发送一个0x00的16进制数据
		let msg = 'hello'
		const buffer = new ArrayBuffer(msg.length)
		const dataView = new DataView(buffer)
		// dataView.setUint8(0, 0)
		for (var i = 0; i < msg.length; i++) {
			dataView.setUint8(i, msg.charAt(i).charCodeAt())
		}
		uni.writeBLECharacteristicValue({
			deviceId: deviceId.value, // 设备ID
			serviceId: '0000FFE0-0000-1000-8000-00805F9B34FB', // 服务UUID
			characteristicId: '0000FFE1-0000-1000-8000-00805F9B34FB', // 特征值
			value: buffer,
			success(res) {
				console.log(res)
			},
			fail(err) {
				console.error(err)
			}
		})
	}

	const read = () => {
		uni.readBLECharacteristicValue({
			deviceId: deviceId.value,
			serviceId: serviceId.value,
			characteristicId: characteristicId.value,
			success(res) {
				console.log('读取指令发送成功')
				console.log(res)
			},
			fail(err) {
				console.log('读取指令发送失败')
				console.error(err)
			}
		})
	}

	const hideDialog = () => {
		dialogshow.value = false;
	}

	watch(blueStatus, (n, o) => {
	})

	onMounted(() => {
		blueConImg.value = blueConImgList[0];
	})
</script>

<style lang="scss">
	.content {
		// background-color: #1E1F24;
		background-color: rgb(28 29 34);
		.content-main {
			// height: 100vh;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			background-size: cover;
			background-color: #1D1E24;
			// background-color: rgb(35 36 44);
			border-top-left-radius: 50rpx;
			border-top-right-radius: 50rpx;
			border-top: 5rpx solid #4F4F5B;
			position: relative;
			
			.bluetooth {
				// padding: 10rpx 0rpx;
				color: white;
				position: absolute;
				left: 0rpx;
				top: 0rpx;
				.bluetooth-con {
					width: 250rpx;
					height: 100rpx;
					.bluetooth-icon {
						padding: 5rpx;
						width: 50rpx;
						height: 50rpx;
					}
					.bluetooth-p {
						font-size: 1rem;
						padding: 5rpx;
						height: 50rpx;
						line-height: 50rpx;
						text-align: center;
					}
				}
			}

			.main-content {
				margin-top: 60rpx;
				.circle {
					width: 500rpx;
					height: 500rpx;
					position: relative;
					border-radius: 50%;
					// left: 200rpx;
					// top: 50rpx;
					box-shadow: inset 0 0 0 5rpx #54c4fd;
					.ab {
						position: absolute;
						left: 0;
						right: 0;
						top: 0;
						bottom: 0;
						margin: auto;
					}
					.circle-left {
						border: 5rpx solid #546063;
						border-radius: 50%;
						clip: rect(0,250rpx,500rpx,0);
					}
					.circle-right {
						border: 5rpx solid #546063;
						border-radius: 50%;
						clip: rect(0,500rpx,500rpx,250rpx);
					}
				}
			}

			.text-area {
				display: flex;
				justify-content: center;

				.title {
					font-size: 36rpx;
					// color: #8f8f94;
					color: white;
				}
			}

			.dialog {
				position: fixed;
				top: 0;
				right: 0;
				bottom: 0;
				left: 0;
				z-index: 1050;
				display: none;
				opacity: 0;
				overflow: hidden;
				-webkit-overflow-scrolling: touch;
				outline: 0;
				background-color: rgba(0, 0, 0, .2);

				.modal-dialog {
					height: 100%;
					width: 100%;
					display: flex;
					align-items: center;
					justify-content: center;

					.dialog-body {
						width: 600rpx;
						background-color: aliceblue;
						border-radius: 8rpx;
						text-align: center;
						padding: 10rpx;

						.dialog-header {
							color: black;
						}

						.dialog-content {
							padding: 10rpx 0rpx;
							font-size: 1.1rem;
							width: 200px;
							height: 200px;

							.blue-box {
								position: relative;
								overflow: hidden;

								.blue-list {
									position: absolute;
									left: 0;
									overflow-x: hidden;
									overflow-y: scroll;

									.blueinfo {
										padding: 5rpx 0rpx;
									}
								}

								.blue-list::-webkit-scrollbar {
									display: none;
								}
							}
						}
					}

				}
			}

			.in {
				display: block;
				opacity: 1;
			}
		}


	}
</style>
