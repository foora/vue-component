<template>
	<nav class="pagination" v-if="showPagination">
		<ul>
			<li>
				<span>共{{total}}条</span>
			</li>
			<li>
				<span class="select">
    <select v-model="pageSize">
        <option v-for="size in pageSizes"  :value='size'>{{size}}条/页</option>
    </select>
    </span>
			</li>
			<li>
				<a :class="[currentPage<=1?disableClass:'',buttonClass]" v-show="totalPage>5" @click="currentPage=1">&lt&lt</a>
			</li>
			<li>
				<a :class="[currentPage<=1?disableClass:'',buttonClass]" v-show="totalPage>1" @click="currentPage=currentPage-1">&lt</a>
			</li>
			<li v-for="page in currentPages">
				<a :class="[page===currentPage?activeClass:'',buttonClass]" @click="currentPage=page">{{page}}</a>
			</li>
			<li>
				<a :class="[currentPage>=totalPage?disableClass:'',buttonClass]" v-show="totalPage>1" @click="currentPage=currentPage+1">&gt</a>
			</li>
			<li>
				<a :class="[currentPage>=totalPage?disableClass:'',buttonClass]" v-show="totalPage>5" @click="currentPage=totalPage">&gt&gt</i></a>
			</li>
			<li>
				前往
				<input type="text" class="jumpInput" :value='currentPage' @keyup.enter="jumpPage($event)" @blur="jumpPage($event)"> 页
			</li>
		</ul>
	</nav>
</template>
<script>
	export default {
		props: {
			size: {
				type: Number,
				default: 10
			},
			total: {
				type: Number
			},
			nowPage: {
				type: Number,
				default: 1
			},
			pageSizes: {
				type: Array,
				default () {
					return [10, 20, 30, 40, 50]
				}
			}
		},

		data() {
			return {
				// 按钮样式class
				disableClass: 'is-disabled',

				activeClass: 'is-primary',
				buttonClass: 'button',
				// 数据
				pageSize: this.size,
				currentPage: this.nowPage
			}
		},

		methods: {
			jumpPage: function (event) {
				let newPage = Number(event.target.value)
				if (isNaN(newPage)) return
				this.currentPage = newPage > this.totalPage ? this.totalPage : newPage < 1 ? 1 : newPage;
			}
		},
		computed: {
			showPagination: function () {
				return this.total ? true : false;
			},
			totalPage: function () {
				return this.total / this.pageSize == 0 ? this.total / this.pageSize : Math.ceil(this.total / this.pageSize);
			},
			currentPages: function () {
				this.currentPage = this.currentPage < 1 ? 1 : this.currentPage > this.totalPage ? this.totalPage : this.currentPage;
				if (this.totalPage < 5) {
					let pageArray = [];
					for (let i = 1; i <= this.totalPage; i++) {
						pageArray.push(i);
					}
					return pageArray;
				}
				if (this.currentPage === 1 || this.currentPage === 2) {
					return [1, 2, 3, 4, 5]
				} else if (this.currentPage === this.totalPage - 1 || this.currentPage === this.totalPage) {
					return [this.totalPage - 4, this.totalPage - 3, this.totalPage - 2, this.totalPage - 1, this.totalPage]
				} else {
					return [this.currentPage - 2, this.currentPage - 1, this.currentPage, this.currentPage + 1, this.currentPage +
						2
					]
				}
			}
		},
		watch: {
			currentPage: function () {
				this.$emit('pageChange', this.currentPage)
			},
			pageSize: function () {
				this.currentPage = 1;
				this.$emit('sizeChange', this.pageSize)
			}
		}
	}
</script>

<style scoped>
	.jumpInput {
		width: 50px;
		height: 26px;
		text-align: center;
		line-height: 26px;
		font-size: 16px;
		border-radius: 3px;
	}
</style>