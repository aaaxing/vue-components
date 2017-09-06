<!--created by Xingrui Ni at 2017/8/19-->
<!--需要传入参数title  titleContent v-model是选择的值输出到外部组件的方式 -->
<template>
	<div class="container" tabindex='0' @keydown.up='allKeyDown(1)' @keydown.down='allKeyDown(2)' @keydown.enter='allKeyDown(3)'>
		<div class='input'>
			<label>{{titleName}}</label>
			<input ref="focus" :placeholder="titleContent" v-model='finalValue' @input='speedSearch' @blur='hideOrder' @focus='showOrder' @click='showOrder' />
		</div>
		<div v-show="visible" class='orderChoose' @mousedown='dealFocus'>
			<div class='title'>可直接选择城市或输入城市全拼简拼和汉字搜索...</div>
			<div class='orderNav'>
				<ul>
					<li v-for='(item,index) in chooseItem' @click="changeOrder(index)" :class="{'orderActive':orderShow==index,'':orderShow!=index}">{{item}}</li>
				</ul>
			</div>
			<div class="orderContainer" ref='scroll'>
				<div class="itemBox" v-show='orderShow==0'>
					<h4>{{'热门选择(成都市)'}}</h4>
					<ul>
						<span v-for='item in hot' @click='choosePlace(item)'>
							&nbsp;{{item.zname}}
						</span>
					</ul>
				</div>
				<div class="itemBox" v-show='orderShow==1'>
					<ul>
						<li v-for='item in groupA'>
							<h3>{{item[0].zname}}</h3>
							<div>
								<span v-for='(items,index) in item' @click='choosePlace(item,items)'>{{items.zname}}</span>
							</div>
						</li>
					</ul>
				</div>
				<div class="itemBox" v-show='orderShow==2'>
					<ul>
						<li v-for='item in groupB'>
							<h3>{{item[0].zname}}</h3>
							<div>
								<span v-for='(items,index) in item' @click='choosePlace(item,items)'>{{items.zname}}</span>
							</div>
						</li>
					</ul>
				</div>
				<div class="itemBox" v-show='orderShow==3'>
					<ul>
						<li v-for='item in groupC'>
							<h3>{{item[0].zname}}</h3>
							<div>
								<span v-for='(items,index) in item' @click='choosePlace(item,items)'>{{items.zname}}</span>
							</div>
						</li>
					</ul>
				</div>
				<div class="itemBox" v-show='orderShow==4'>
					<ul>
						<li v-for='item in groupD'>
							<h3>{{item[0].zname}}</h3>
							<div>
								<span v-for='(items,index) in item' @click='choosePlace(item,items)'>{{items.zname}}</span>
							</div>
						</li>
					</ul>
				</div>
			</div>
		</div>
		<div v-show='speedVisible' class='speedSearch' @mousedown='dealFocus'>
			<ul>
				<div class='head' v-show='!nonePlace'>
					<div>
						<button class="button" @click='whichPage(1)' :disabled="indexPage==0">上一页</button>
					</div>
					<div style="line-height: 34px;">{{indexPage+1}}/{{pageShow}}</div>
					<div>
						<button class='button' @click='whichPage(2)' :disabled="(indexPage + 1) * 8 >= speedLoad.length">下一页</button>
					</div>
				</div>
				<div class='searchNone' v-show='nonePlace'>没有搜索到相应地址</div>
				<li v-for='(item,index) in speedLoadShow' :class='{"itemActive":index==indexItem,"":index!=indexItem}' @mouseenter="indexItem = index" @mouseleave="indexItem = 0" @click="clickChoose(index)">
					&nbsp;&nbsp;&nbsp;{{item.showName}}
					<p><span>{{item.keyWords}}</span>{{item.lastWords}}</p>
				</li>
			</ul>
		</div>
	</div>
</template>

<script>
	import place from '../../commons/data/townplace.js'; //数据源
	export default {
		data() {
			let chooseItem = ["热门选择", "ABCDEF", "GHIJKL", "MNOPQRS", "TUVWXYZ"]; //选项卡列表
			//以下为控制状态变量
			let visible = false; //列表搜索显示状态
			let speedVisible = false; //快速搜索显示状态
			let orderShow = 0; //列表选择选项卡
			let focusShow = false; //控制点击组件外位置关闭搜索窗口
			let indexPage = 0; //快速查询的当前页码
			let indexItem = 0; //快速查询的当前选择条目
			//以下为数据处理存储结果（DOM操作用）
			let hot = []; //热门搜索显示数据（成都范围内）
			let groupA = []; //ABCDEF显示数据
			let groupB = []; //GHIJKL显示数据
			let groupC = []; //MNOPQRS显示数据
			let groupD = []; //TUVWXYZ显示数据
			let finalValue = ''; //绑定到表单的显示数据，字符串，这只是显示数据，不是传给后台的最后数据
			let speedLoad = []; //快速查询搜索匹配到的所有数据
			let speedLoadShow = []; //快速查询当前页面的显示数据
			return {
				chooseItem,
				visible,
				speedVisible,
				orderShow,
				focusShow,
				indexPage,
				indexItem,
				hot,
				groupA,
				groupB,
				groupC,
				groupD,
				finalValue,
				speedLoad,
				speedLoadShow
			}
		},
		components: {},
		props: {
			value: { // 必须要使用value	　　　　　
				default: '',
			}
		},
		computed: {
			pageShow() { //判断上一页下一页是否满足，不满足禁用button
				if(this.speedLoad.length != 0) {
					return Math.ceil(this.speedLoad.length / 8)
				} else if(this.speedLoad.length == 0) {
					return '1'
				}
			},
			nonePlace() { //没有匹配到查询结果，提示
				if(this.speedLoad.length == 0) {
					return true
				} else {
					return false
				}
			}
		},
		mounted() {
			this.initGroups(place.place); //数据初始化
			window.addEventListener('mousedown', () => { //点击组件外部关闭组件所有状态，并且自动补全后判断选择的值是否有效
				this.focusShow = false;
				if(this.speedVisible) {
					if(this.speedLoadShow.length) {
						this.$emit('input', this.speedLoadShow[0])
						this.finalValue = this.speedLoadShow[0].zname
					} else {
						this.$emit('input', null)
						this.finalValue = null
					}
				}
			})
		},
		props: ['titleName', 'titleContent'],
		methods: {
			//如果需要进行组件复用，改变的只是数据，所以只用关心前两个处理数据的函数和页面布局的一些修改就行
			//数据与状态初始化
			initGroups(place) { //初始化列表查询数据
				let dealInfo = [
					[]
				];
				let count = 0;
				let gA = [];
				let gB = [];
				let gC = [];
				let gD = [];
				for(let i in place) {
					if(place[i].zname == '市辖区') { //去除所有市辖区,并且以lev=2来进行字母排序
						place.splice(i, 1);
					} else {
						if(place[i].lev == 2) {
							count++;
							dealInfo[count] = [];
						}
						if(place[i].lev < 4) {
							dealInfo[count].push(place[i]);
						}
					}
				}
				//解决深浅拷贝，将部分数据仅仅用来展示热门选择
				let hotInfo = JSON.stringify(dealInfo[1]);
				hotInfo = JSON.parse(hotInfo);
				this.hot = hotInfo;
				//遍历根据字母顺序进行分组，将lev=2的地方及其子地方排序后，放入相应的展示区
				for(let i = 0; i < dealInfo.length; i++) {
					for(let j = i; j < dealInfo.length; j++) {
						if(dealInfo[i][0].fs[0] > dealInfo[j][0].fs[0]) {
							let record = dealInfo[i];
							dealInfo[i] = dealInfo[j];
							dealInfo[j] = record;
						}
					}

					if(dealInfo[i][0].fs[0].toUpperCase() >= 'A' && dealInfo[i][0].fs[0].toUpperCase() <= 'F') {
						gA.push(dealInfo[i]);
					} else if(dealInfo[i][0].fs[0].toUpperCase() >= 'G' && dealInfo[i][0].fs[0].toUpperCase() <= 'L') {
						gB.push(dealInfo[i]);
					} else if(dealInfo[i][0].fs[0].toUpperCase() >= 'M' && dealInfo[i][0].fs[0].toUpperCase() <= 'S') {
						gC.push(dealInfo[i]);
					} else if(dealInfo[i][0].fs[0].toUpperCase() >= 'T' && dealInfo[i][0].fs[0].toUpperCase() <= 'Z') {
						gD.push(dealInfo[i]);
					} else {}
				}
				//进行分组
				this.groupA = gA;
				this.groupB = gB;
				this.groupC = gC;
				this.groupD = gD;
			},
			//组装快速查询需要的所有数据
			matchingSpeedList(allItem) {
				for(let i in allItem) {
					allItem[i].showName = '';
					if(allItem[i].fs.indexOf(this.finalValue.toLowerCase()) == 0) {
						//按照全拼进行查找（1234根据等级向前查询父地点）
						switch(allItem[i].lev) {
							case 1:
								allItem[i].showName = allItem[i].zname;
								allItem[i].keyWords = this.finalValue;
								allItem[i].lastWords = allItem[i].fs.substr(this.finalValue.length);
								break;
							case 2:
								allItem[i].showName = allItem[i].zname;
								allItem[i].keyWords = this.finalValue;
								allItem[i].lastWords = allItem[i].fs.substr(this.finalValue.length);
								break;
							case 3:
								for(let j = i; j > 0; j--) {
									if(allItem[j].lev == 2) {
										allItem[i].showName = allItem[j].zname + '-' + allItem[i].zname;
										allItem[i].keyWords = this.finalValue;
										allItem[i].lastWords = allItem[i].fs.substr(this.finalValue.length);
										break;
									}
								};
								break;
							case 4:
								for(let j = i; j > 0; j--) {
									if(allItem[j].lev == 3) {
										allItem[i].showName = allItem[j].zname + '-' + allItem[i].zname;
										for(let k = j; k > 0; k--) {
											if(allItem[k].lev == 2) {
												allItem[i].showName = allItem[k].zname + '-' + allItem[i].showName;
												allItem[i].keyWords = this.finalValue;
												allItem[i].lastWords = allItem[i].fs.substr(this.finalValue.length);
												break;
											}
										}
										break;
									}
								};
								break;
							default:
								;
								break;
						}
						this.speedLoad.push(allItem[i]);
					} else if(allItem[i].as.indexOf(this.finalValue.toLowerCase()) == 0) {
						//按照简写进行查找（1234根据等级向前查询父地点）
						switch(allItem[i].lev) {
							case 1:
								allItem[i].showName = allItem[i].zname;
								allItem[i].keyWords = this.finalValue;
								allItem[i].lastWords = allItem[i].as.substr(this.finalValue.length);
								break;
							case 2:
								allItem[i].showName = allItem[i].zname;
								allItem[i].keyWords = this.finalValue;
								allItem[i].lastWords = allItem[i].as.substr(this.finalValue.length);
								break;
							case 3:
								for(let j = i; j > 0; j--) {
									if(allItem[j].lev == 2) {
										allItem[i].showName = allItem[j].zname + '-' + allItem[i].zname;
										allItem[i].keyWords = this.finalValue;
										allItem[i].lastWords = allItem[i].as.substr(this.finalValue.length);
										break;
									}
								};
								break;
							case 4:
								for(let j = i; j > 0; j--) {
									if(allItem[j].lev == 3) {
										allItem[i].showName = allItem[j].zname + '-' + allItem[i].zname;
										for(let k = j; k > 0; k--) {
											if(allItem[k].lev == 2) {
												allItem[i].showName = allItem[k].zname + '-' + allItem[i].showName;
												allItem[i].keyWords = this.finalValue;
												allItem[i].lastWords = allItem[i].as.substr(this.finalValue.length);
												break;
											}
										}
										break;
									}
								};
								break;
							default:
								;
								break;
						}
						this.speedLoad.push(allItem[i]);
					} else if(allItem[i].zname.indexOf(this.finalValue) == 0) {
						//按照汉字进行查找（1234根据等级向前查询父地点）
						switch(allItem[i].lev) {
							case 1:
								allItem[i].showName = allItem[i].zname;
								allItem[i].keyWords = this.finalValue;
								allItem[i].lastWords = allItem[i].zname.substr(this.finalValue.length);
								break;
							case 2:
								allItem[i].showName = allItem[i].zname;
								allItem[i].keyWords = this.finalValue;
								allItem[i].lastWords = allItem[i].zname.substr(this.finalValue.length);
								break;
							case 3:
								for(let j = i; j > 0; j--) {
									if(allItem[j].lev == 2) {
										allItem[i].showName = allItem[j].zname + '-' + allItem[i].zname;
										allItem[i].keyWords = this.finalValue;
										allItem[i].lastWords = allItem[i].zname.substr(this.finalValue.length);
										break;
									}
								};
								break;
							case 4:
								for(let j = i; j > 0; j--) {
									if(allItem[j].lev == 3) {
										allItem[i].showName = allItem[j].zname + '-' + allItem[i].zname;
										for(let k = j; k > 0; k--) {
											if(allItem[k].lev == 2) {
												allItem[i].showName = allItem[k].zname + '-' + allItem[i].showName;
												allItem[i].keyWords = this.finalValue;
												allItem[i].lastWords = allItem[i].zname.substr(this.finalValue.length);
												break;
											}
										}
										break;
									}
								};
								break;
							default:
								;
								break;
						}
						this.speedLoad.push(allItem[i]);
					}
				}
			},
			//点击表单条目选择
			choosePlace(item, items) {
				this.orderShow = 0; //重置标准选择的按钮，默认到热门选择
				event.stopPropagation();
				let backInfo;
				if(items) {
					backInfo = items.zname;
					this.$emit('input', items);
				} else {
					backInfo = item.zname;
					this.$emit('input', item);
				}
				this.finalValue = backInfo;
				this.visible = false;
				this.speedVisible = false;
				this.indexPage = 0;
				this.indexItem = 0;
				this.focusShow = false;
			},
			//键盘改变input值进行匹配处理函数
			speedSearch() {
				this.orderShow = 0; //重置标准选择的按钮，默认到热门选择
				this.visible = false;
				this.speedVisible = true;
				this.speedLoad = [];
				this.indexPage = 0;
				this.indexItem = 0;
				if(!this.finalValue) {
					this.speedVisible = false;
					return
				}
				let allItem = place.place;
				this.matchingSpeedList(allItem);
				this.speedLoadInfo();
			},
			//快捷查询键盘事件(上下键和enter键)
			allKeyDown(index) {
				event.stopPropagation();
				if(this.speedVisible) {
					switch(index) {
						case 1:
							if(this.indexItem == 0) {
								return
							} else {
								this.indexItem--;
							};
							break;
						case 2:
							if(this.indexItem == this.speedLoadShow.length - 1) {
								return
							} else {
								this.indexItem++;
							};
							break;
						case 3:
							this.finalValue = this.speedLoadShow[this.indexItem].zname;
							this.visible = false;
							this.speedVisible = false;
							this.$emit('input', this.speedLoadShow[this.indexItem]);
							break;
						default:
							;
							break;
					}
				}
			},
			//点击快捷查询上一页和下一页
			whichPage(index) {
				if(index == 1) {
					if(this.indexPage == 0) {
						return
					} else {
						this.indexPage--;
						this.indexItem = 0;
						this.speedLoadInfo();
					}
				} else if(index == 2) {
					if((this.indexPage + 1) * 8 >= this.speedLoad.length) {
						return
					} else {
						this.indexPage++;
						this.indexItem = 0;
						this.speedLoadInfo();
					}
				}
			},
			//动态更新快捷查询列表
			speedLoadInfo() {
				this.speedLoadShow = [];
				for(let i = 8 * this.indexPage; i < 8 * this.indexPage + 8 && i < this.speedLoad.length; i++) {
					this.speedLoadShow.push(this.speedLoad[i]);
				}
			},
			//鼠标悬停选择效果
			hoverChoose(index) {
				this.indexItem = index;
			},
			//鼠标点击选择效果
			clickChoose(index) {
				event.stopPropagation();
				this.finalValue = this.speedLoadShow[index].zname;
				this.$emit('input', this.speedLoadShow[index]);
				this.focusShow = false;
				this.visible = false;
				this.speedVisible = false;
			},
			//隐藏展示表单查询（注意使用mouseDown事件先于input的blur事件发生）
			hideOrder() {
				if(this.focusShow == true) {
					this.$refs.focus.focus();
				} else {
					this.orderShow = 0; //重置标准选择的按钮，默认到热门选择
					this.visible = false;
					this.speedVisible = false;
				}
			},
			//展示表单数据
			showOrder() {
				if(this.visible == false && this.speedVisible == false) {
					this.visible = true;
				} else {
					return
				}
			},
			//点击两个搜索框时，自动聚焦且保持显示状态
			dealFocus() {
				event.stopPropagation();
				this.focusShow = true;
			},
			//点击表单搜索选项卡进行切换，并重置滚动高度
			changeOrder(index){
				this.orderShow=index;
				let order = this.$refs.scroll.getElementsByClassName('itemBox');
				for(let i=0;i<order.length;i++){
					order[i].scrollTop = 0;
				}
			}
		}
	}
</script>

<style scoped>
	.container {
		padding: 0;
		outline: none;
		width: 100%;
		margin-bottom: 5px;
		position: relative;
	}
	
	.input label {
		width: 140px;
		text-align: right;
		vertical-align: middle;
		float: left;
		font-size: 12px;
		color: #495060;
		line-height: 1;
		padding: 10px 12px 10px 0;
		box-sizing: border-box;
	}
	
	.input input {
		width: 200px !important;
		display: inline-block;
		width: 100%;
		height: 32px;
		line-height: 1.5;
		padding: 4px 7px;
		font-size: 12px;
		border: 1px solid #dddee1;
		border-radius: 4px;
		color: #495060;
		background-color: #fff;
		background-image: none;
		position: relative;
		cursor: text;
	}
	
	.input input:focus {
		border-color: #57a3f3;
		box-shadow: 0 0 1px 1px lightblue;
	}
	
	.orderChoose {
		width: 70%;
		position: absolute;
		top: 35px;
		left: 140px;
		-webkit-box-shadow: 0 0 6px darkgray;
		-moz-box-shadow: 0 0 6px darkgray;
		box-shadow: 0 0 6px darkgray;
		background: white;
		z-index: 10000;
		overflow: hidden;
		border-radius: 4px;
	}
	
	.title {
		color: #666;
		line-height: 28px;
		margin: 0 0 0 20px;
		color: #666;
		font-size: 12px !important;
		margin: 0 0 0 10px;
	}
	
	.orderNav {
		width: 100%;
		height: 30px;
	}
	
	.orderNav ul {
		width: 100%;
		height: 100%;
		border-bottom: 1px solid #F6F6F6;
	}
	
	.orderNav ul li {
		float: left;
		cursor: pointer;
		width: 18%;
		margin-left: 1.8%;
		text-align: center;
		line-height: 30px;
		border-radius: 4px 4px 0 0;
		background: #F6F6F6;
		color: gray;
		font-size: 12px !important;
	}
	
	.orderActive {
		color: white !important;
		background: #00aaff !important;
	}
	
	.orderContainer {
		position: relative;
	}
	
	.itemBox {
		height: 260px;
		width: 100%;
		overflow-y: scroll;
		background: white;
	}
	
	.itemBox h4 {
		margin-left: 10px;
		font-size: 13px;
		font-weight: 600;
		color: #2DB7F5;
		line-height: 26px;
		font-weight: 500;
		margin-top: 5px;
	}
	
	.itemBox h3 {
		width: 20%;
		float: left;
		font-size: 13px;
		font-weight: 600;
		line-height: 26px;
		font-weight: 500;
		text-align: center;
		margin-top: 5px;
		color: #2DB7F5
	}
	
	.itemBox:nth-child(1) span {
		display: inline-block;
		width: 20%;
		line-height: 26px;
		padding: 0 5px;
		font-size: 11px;
	}
	
	.itemBox span {
		font-size: 11px;
		display: inline-block;
		width: 25%;
		line-height: 26px;
		padding: 0 5px;
	}
	
	.itemBox span:hover {
		color: #00AAFF;
		cursor: pointer;
	}
	
	.itemBox ul li div {
		width: 80%;
		float: left;
		margin-top: 5px;
	}
	
	.speedSearch {
		cursor: pointer;
		position: absolute;
		top: 35px;
		z-index: 10000;
		min-height: 30px;
		max-height: 300px;
		width: 35%;
		margin-left: 140px;
	}
	
	.speedSearch ul {
		min-height: 30px;
		max-height: 300px;
		width: 140%;
		overflow: hidden;
		background: white;
		border-radius: 4px;
		-webkit-box-shadow: 0 0 6px darkgray;
		-moz-box-shadow: 0 0 6px darkgray;
		box-shadow: 0 0 6px darkgray;
		padding-bottom: 5px;
	}
	
	.speedSearch ul .head {
		border-bottom: 1px solid #E0E0E0;
		margin-bottom: 5px;
		color: black;
		overflow: hidden;
	}
	
	.speedSearch ul .head div {
		width: 33.33333%;
		float: left;
		text-align: center;
		height: 30px;
		line-height: 30px;
		font-size: 10px;
	}
	
	.button {
		height: 26px;
		width: 45px;
		border-radius: 4px;
		line-height: 26px !important;
		margin-top: 2px;
		outline: none;
		border: 1px solid #DDDEE1;
		color: gray;
		background: white;
	}
	
	.button:hover {
		color: #00AAFF;
		border: 1px solid #00AAFF;
	}
	
	.button:disabled {
		cursor: not-allowed;
		background: #F7F7F7;
		color: gray;
		border: 1px solid #DDDEE1;
	}
	
	.speedSearch ul li {
		width: 95%;
		margin-left: 2.5%;
		position: relative;
		font-size: 11px;
		line-height: 28px;
	}
	
	.speedSearch ul li p {
		position: absolute;
		right: 5px;
		top: 0;
		height: 100%;
	}
	
	.speedSearch ul li p span {
		color: red;
	}
	
	.itemActive {
		color: #00AAFF;
		background: aliceblue;
	}
	
	.searchNone {
		text-align: center;
		font-size: 13px;
		line-height: 30px;
	}
	
	input::-webkit-input-placeholder {
		color: #BBBEC4;
	}
	
	input:-moz-placeholder {
		color: #BBBEC4;
	}
	
	input::-moz-placeholder {
		color: #BBBEC4;
	}
	
	input:-ms-input-placeholder {
		color: #BBBEC4;
	}
</style>
<style>
	.noClose .ivu-select-dropdown {
		padding: 0;
		border: 1px solid #66afe9;
	}
</style>