<template>
	<view v-if="showselected">
		<view class="tabbar">
			<view class="navigator">
				<view class="navigator-item" v-for="(item,index) in tabBar.list" :key="item.pagePath" \
					@click="switchTab(index,item)">
					<img src="item.iconPath" class="icon" v-if="selectedIndex !== index">
					<img src="item.selectedIconPath" class="icon" v-else>
					<text :class="['item-text',{'text-active':selectedIndex === index}]">
						{{item.text}}
					</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import {ref} from 'vue'


const selectedIndex = ref(0);
const showselected = ref(false);

const tabBar = ref([
	"pagePath": "pages/index/index",
	"iconPath": "static/images/",
	
])

</script>

<style lang="scss">

</style>