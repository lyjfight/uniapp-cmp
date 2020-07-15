<template>
	<view :class="classes">
		<view class="li">
			<text :class="arrowClasses" @click="handleExpand">
				<view style="padding: 5rpx;display: inline-block;">
					<uni-icons v-if="showArrow" :type="arrowType" size="24"></uni-icons>
				</view>
				<view style="padding: 5rpx;display: inline-block;" class="tree-load-loop">
					<uni-icons v-if="showLoading" type="reload" size="24"></uni-icons>
					<!-- <view class="btn-loading" v-if="showLoading" style="width: 16px;height: 16px;"></view> -->
				</view>
			</text>
			<text :class="titleClasses" @click="handleSelect">
				{{treeData ? treeData.title : ''}}
			</text>
			<Tree-node
				class="ul"
				v-if="treeData.expand"
				v-for="(item, i) in children"
				:key="i"
				:data="item"
				:children-key="childrenKey">
			</Tree-node>
		</view>
	</view>
</template>

<script>
	/* children:
		title	标题	String | Element String	-
		expand	是否展开直子节点	Boolean	false
		children	子节点属性数组	Array	-
	 */
	import TreeNode from './node.vue';
	import uniIcons from '@/components/uni-icons/uni-icons.vue'
	
	const prefixCls = 'lyj-tree';
	
	function findComponentUpward (context, componentName, componentNames) {
	    if (typeof componentName === 'string') {
	        componentNames = [componentName];
	    } else {
	        componentNames = componentName;
	    }
	
	    let parent = context.$parent;
	    let name = parent.$options.name;
	    while (parent && (!name || componentNames.indexOf(name) < 0)) {
	        parent = parent.$parent;
	        if (parent) name = parent.$options.name;
	    }
	    return parent;
	}
	
	export default {
		name: 'TreeNode',
		components: {
			TreeNode,
			uniIcons
		},
		inject: ['TreeInstance'],
		props: {
			data: {
				type: Object,
				default () {
					return {};
				}
			},
			childrenKey: {
				type: String,
				default: 'children'
			},
		},
		data() {
			return {
				prefixCls: prefixCls,
				treeData: null
			}
		},
		created() {
			this.treeData = {...this.data}
		},
		computed: {
			classes () {
				return [
					`${prefixCls}-children`
				];
			},
			arrowClasses () {
				return [
					`${prefixCls}-arrow`,
					{
						[`${prefixCls}-arrow-open`]: this.treeData.expand
					}
				];
			},
			titleClasses () {
				return [
					`${prefixCls}-title`
				];
			},
			showLoading () {
				return 'loading' in this.treeData && this.treeData.loading;
			},
			children () {
				return this.treeData[this.childrenKey];
			},
			showArrow () {
				return (this.treeData[this.childrenKey] && this.treeData[this.childrenKey].length) || ('loading' in this.treeData && !this.treeData.loading);
			},
			arrowType () {
				if(this.treeData.expand) {
					return 'arrowdown'
				}else {
					return 'arrowright'
				}
			}
		},
		methods: {
			handleExpand() {
				const item = this.treeData
				
				if (item[this.childrenKey].length === 0) {
					const tree = findComponentUpward(this, 'Tree');
					// console.log(this.TreeInstance.$emit('load-data'))
					if('loading' in item && !item.loading) { //异步加载
						this.$set(this.treeData, 'loading', true);
						tree.$emit('loadData', item, children => {
								this.$set(this.treeData, 'loading', false);
								if (children.length) {
										this.$set(this.treeData, this.childrenKey, children);
										this.$nextTick(() => this.handleExpand());
								}else {
									delete this.treeData.loading
									delete this.treeData[this.childrenKey]
								}
						})
						return
					}
					// if (tree && tree.loadData) {
					// 	this.$set(this.treeData, 'loading', true);
					// 	tree.loadData(item, children => {
					// 			this.$set(this.treeData, 'loading', false);
					// 			if (children.length) {
					// 					this.$set(this.treeData, this.childrenKey, children);
					// 					this.$nextTick(() => this.handleExpand());
					// 			}
					// 	});
					// 	return;
					// }
				}
				if(item[this.childrenKey] && item[this.childrenKey].length) {
					this.$set(this.treeData, 'expand', !this.treeData.expand);
				}
			},
			handleSelect() {
				const tree = findComponentUpward(this, 'Tree');
				tree.$emit('on-select', this.treeData)
			}
		}
	}
</script>

<style scoped>
	.lyj-tree-children .li .lyj-tree-children {
		/* padding: 0 0 0 18px; */
		padding: 0 0 0 32px;
	}
	.lyj-tree-children .li {
		margin: 8px 0;
	}
	.lyj-tree-title {
		font-size: 30rpx;
	}
	.btn-loading {
		border-top-color:rgb(0, 0, 0);
		border-right-color:rgb(0, 0, 0);
		border-left-color:rgb(0, 0, 0);
		border-width: 1px;
		display: inline-block;
	}
</style>
