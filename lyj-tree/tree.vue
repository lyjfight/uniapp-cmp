<template>
	<view :class="prefixCls">
		<Tree-node
			v-for="(item, i) in stateTree"
			:key="i"
			:data="item"
			:children-key="childrenKey">
		</Tree-node>
		<view :class="[prefixCls + '-empty']" v-if="!stateTree.length">{{ localeEmptyText }}</view>
	</view>
</template>

<script>
	import TreeNode from './node.vue';
	
	const prefixCls = 'lyj-tree';
	
	export default {
		name: 'Tree',
		components: { TreeNode },
		provide () {
			return { TreeInstance: this };
		},
		props: {
			data: {
				type: Array,
				default () {
					return [];
				}
			},
			emptyText: {
				type: String
			},
			// loadData: {
			// 	type: Function
			// },
			childrenKey: {
				type: String,
				default: 'children'
			}
		},
		data() {
			return {
				prefixCls: prefixCls,
				stateTree: this.data,
			}
		},
		watch: {
			data: {
				deep: true,
				handler () {
					this.stateTree = this.data;
				}
			}
		},
		computed: {
			localeEmptyText () {
				if (typeof this.emptyText === 'undefined') {
					return 'Пе';
				} else {
					return this.emptyText;
				}
			}
		}
	}
</script>

<style>
</style>
