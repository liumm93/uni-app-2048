<template>
	<view class="content">
		<view class="memu">
			<view @tap="init">再来一次</view>
			<view>{{max_block}}</view>
		</view>
		<view class="maps" @touchstart="touchStart" @touchend="touchEnd">
			<view :class="classObject(item)" v-for="(item, index) in map_obj"
				:key="'item-' + index">
				<view v-show="item">{{item}}</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				size: 4,
				map_obj: {},
				max_block: 2,
				empty_block: 0,
				game_over: false,
				client_x: 0,
				client_y: 0,
				style_list: {
					'2': 'item-2',
					'4': 'item-4',
					'8': 'item-8',
					'16': 'item-16',
					'32': 'item-32',
					'64': 'item-64',
					'128': 'item-128',
					'256': 'item-256',
					'512': 'item-512',
					'1024': 'item-1024',
				},
			}
		},
		onLoad() {
			this.init();
		},
		methods: {
			classObject: function(val) {
				return {
					block: true,
					['item-' + val]: true,
				}
			},
			
			init() {
				this.initMaps();
				this.addBlock();
				this.addBlock();
			},

			initMaps() {
				this.game_over = false;
				this.max_block = 2;
				this.map_obj = {};
				this.empty_block = this.size * this.size;
				let key = '';
				for (let i = 0; i < this.size; i++) {
					for (let j = 0; j < this.size; j++) {
						key = `${i}-${j}`;
						this.map_obj[key] = 0;
					}
				}
			},

			addBlock() {
				if (this.max_block == 1024) {
					uni.showToast({
						title: '你赢了',
						icon: 'none',
						duration: 1000,
					});
					this.game_over = true;
					return;
				}
				
				if (this.empty_block == 0) {
					this.game_over = true;
				}
				
				if (this.game_over) {
					uni.showToast({
						title: '游戏结束',
						icon: 'none',
						duration: 1000,
					});
					return;
				}
				
				let [x, y] = [Math.floor(Math.random() * this.size), Math.floor(Math.random() * this.size)];
				let key = `${x}-${y}`;
				if (this.map_obj[key] == 0) {
					this.$set(this.map_obj, key, 2);
					this.empty_block -= 1;
				} else {
					this.addBlock();
				}
			},

			touchStart(e) {
				this.client_x = e.changedTouches[0].clientX;
				this.client_y = e.changedTouches[0].clientY;
			},

			touchEnd(e) {
				if	(this.game_over) {
					return;
				}
				
				const distance_x = e.changedTouches[0].clientX - this.client_x;
				const distance_y = e.changedTouches[0].clientY - this.client_y;

				if (Math.abs(distance_x) > Math.abs(distance_y)) {
					if (distance_x >= 30) {
						// 右滑
						this.moveRight();
					} else if (distance_x <= -30) {
						// 左滑
						this.moveLeft();
					}
				} else {
					if (distance_y >= 30) {
						// 下滑
						this.moveBottom();
					} else if (distance_y <= -30) {
						// 上滑
						this.moveTop();
					}
				}
				
				this.addBlock();
			},

			moveTop() {
				let [key, next_key, has_move] = ['', '', false];

				for (let i = this.size - 1; i > 0; i--) {
					for (let j = 0; j < this.size; j++) {
						key = `${i}-${j}`;
						next_key = `${i - 1}-${j}`;
						this.doMove(key, next_key);
					}
				}
				this.$forceUpdate();
			}, 
			
			moveBottom() {
				let [key, next_key, has_move] = ['', '', false];
			
				for (let i = 0; i < this.size - 1; i++) {
					for (let j = 0; j < this.size; j++) {
						key = `${i}-${j}`;
						next_key = `${i + 1}-${j}`;
						this.doMove(key, next_key);
					}
				}
				this.$forceUpdate();
			},
			
			moveLeft() {
				let [key, next_key, has_move] = ['', '', false];
			
				for (let i = 0; i < this.size; i++) {
					for (let j = this.size - 1; j > 0; j--) {
						key = `${i}-${j}`;
						next_key = `${i}-${j - 1}`;
						this.doMove(key, next_key);
					}
				}
				this.$forceUpdate();
			},
			
			moveRight() {
				let [key, next_key, has_move] = ['', '', false];
			
				for (let i = 0; i < this.size; i++) {
					for (let j = 0; j < this.size - 1; j++) {
						key = `${i}-${j}`;
						next_key = `${i}-${j + 1}`;
						this.doMove(key, next_key);
					}
				}
				this.$forceUpdate();
			}, 
			
			doMove(key, next_key) {
				if (this.map_obj[key] != 0) {
					if (this.map_obj[next_key] === 0) {
						this.map_obj[next_key] = this.map_obj[key];
						this.map_obj[key] = 0;
					} else if (this.map_obj[next_key] == this.map_obj[key]) {
						this.map_obj[next_key] += this.map_obj[key];
						this.map_obj[key] = 0;
						this.empty_block += 1;
					}
					
					this.max_block = Math.max(this.max_block, this.map_obj[next_key]);
				}
			}
		}
	}
</script>

<style>
	.content {
		width: 280px;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		margin: auto;
	}
	
	.memu {
		width: 100%;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-around;
		margin-top: 10px;
	}
	
	.maps {
		width: 100%;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		align-items: center;
		margin-top: 10px;
		padding-bottom: 10px;
		padding-right: 10px;
		background-color: #F1F1F1;
	}

	.block {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 60px;
		height: 60px;
		margin-top: 10px;
		margin-left: 10px;
		background-color: #F8F8F8;
	}

	.item-2 {
		background-color: #FFFFFF;
	}

	.item-4 {
		background-color: #FFF8DC;
	}

	.item-8 {
		background-color: #FAEBD7;
	}

	.item-16 {
		background-color: #FFE4C4;
	}

	.item-32 {
		background-color: #F5DEB3;
	}

	.item-64 {
		background-color: #DEB887;
	}

	.item-128 {
		background-color: #FFA07A;
	}

	.item-256 {
		background-color: #FFA500;
	}

	.item-512 {
		background-color: #F4A460;
	}

	.item-1024 {
		background-color: #FF8C00;
	}
</style>
