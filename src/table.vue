<template>
   <table class="table is-bordered is-striped is-narrow" v-if="showTable">
            <thead>
              <tr>
                <th v-if='toHideColumns.length>0' class='is-icon'></th>
                <th v-if="checkbox" class="table-checkbox"><input v-model="isAllChecked"  type="checkbox" @change="checkAllToggle(isAllChecked)"/></th>
                <template v-for="column in columns">
                <th v-if="column.show && column.key !== '_actions'">{{column.label}}</th>
                <th v-if="column.key === '_actions'" :colspan='actions.length'>{{column.label}}</th>
                </template>
              </tr>
            </thead>
            <tbody>
              <template v-for='(item,index) in items'>
              <tr>
                <td v-if='toHideColumns.length>0' @click='toHideColumns.length>0 ? showHideColumnsToggle(index) : null' class='is-icon'><i :class="showHideColumnsIndexes[index]?'fa fa-chevron-circle-down':'fa fa-chevron-circle-right'"></i></td>
                <td v-if="checkbox" class="table-checkbox"> <input type="checkbox" :value='index' v-model="checkedIndexes"/></td>
                <td v-for="column in toShowColumns">
                  {{item[column.key]|dictFilter(column.key,dict)}}
                </td>
                <td v-if="actions" v-for="action in actions" class="is-icon">
                    <a :class='action.class' @click='handler(action.name,index)'><i v-if="action.icon" :class='action.icon'></i>{{action.label}}</a>
                </td>
              </tr>
              <transition name='slide-fade'>
              <tr  v-show='showHideColumnsIndexes[index]' v-if='toHideColumns.length>0'>
                <td :colspan= 'actions.length + toShowColumns.length + 2'>
                <ul>
                  <li v-for="column in toHideColumns"><span class='hideColumnsLabel'>{{column.label}}</span>:&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp{{item[column.key]|dictFilter(column.key,dict)}}</li>
                </ul>
                </td>
              </tr>
              </transition>
              </template>
            </tbody>
          </table>
</template>

<script>
export default {
  props: {
    checked: { 
      // 当前数据项中需要显示的勾选状态
      type: Array,
      default() {
        return []
      }
    },
    checkbox: {
      // 是否显示勾选框
      type: Boolean,
      default: false
    },
    items: {
      // 数据项
      type: Array
    },
    columns: {
      // 表格显示的数据列（包含显示和隐藏的）
      type: Array
    },
    actions: {
      // 表格需要的操作
      type: Array
    },
    dict: {
      // 数据字典
      type: Object,
      default() {
        return {}
      }
    }
  },
  data() {
    return {
      checkedIndexes: [], // 组件内部的勾选状态
      isAllChecked: false, // 用于判断是否全部勾选了
      showHideColumnsIndexes: [] // 用于存储表格各行隐藏起来的数据显示情况
    }
  },
  methods: {
    showHideColumnsToggle: function(index) {
      // 切换隐藏列表的显示与隐藏
      if(!this.showHideColumnsIndexes[index]) {
        this.$set(this.showHideColumnsIndexes, index, true);
      } else {
        this.$set(this.showHideColumnsIndexes, index, false);
      }
    },
    handler: function(action, index) {
      // 触发相应的操作按钮的事件
      this.$emit(action, index)
    },
    checkAllToggle: function(isAllChecked) {
      // 切换全部勾选和全部不勾选
      if(!isAllChecked) {
        this.checkedIndexes = []
      } else {
        this.checkedIndexes = this.items.map(function(item, index) {
        return index
        })
      }
    }
  },
  computed: {
    toShowColumns: function() {
      // 要显示在表格内的列
      return this.columns.filter(function(column) {
        return column.key !== '_actions' && column.show
      })
    },
    toHideColumns: function() {
     // 需要可隐藏的列
      return this.columns.filter(function(column) {
        return column.key !== '_actions' && !column.show
      })
    },
    showTable: function() {
      // 根据是否有数据判断是否显示表格
      return this.items.length ? true : false
    }
  },
  filters: {
      dictFilter: function(value, key, dict) {
          // 把数据字典中的内容对应到数据中去 
          if(!value)return''
          if(!dict.hasOwnProperty(key))return value
          else return dict[key][value]
      }
  },
  watch: {
    items: function() {
      // 观察外部传入的数据项是否修改了
      this.checkedIndexes = this.checked // 数据项修改时要根据外部提供的信息重置当前的勾选状态
      this.showHideColumnsIndexes = []; // 数据项修改时把所以打开的隐藏列表收回去
    },
    checkedIndexes: function() {
      // 观察表格的勾选状态是否修改了，修改了后告诉外部现在的勾选状态
      this.isAllChecked=this.checkedIndexes.length===this.items.length?true:false // 全部勾选的话，要把全选的勾选框对应过来
      this.$emit('checkedChange', this.checkedIndexes)
    }
  }
}
</script>

<style scoped>
  .table-checkbox {
    text-align: center;
    vertical-align: middle;
    font-size: 18px;
    line-height: 100%;
    width: 20px;
  }
  .table-checkbox input[type="checkbox"] {
    cursor:pointer;
    height:1.5rem;
    width:1.5rem;
  }
  .hideColumnsLabel {
    font-weight: 700;
    font-size: 16px;
  }
  .slide-fade-enter-active {
    animation: slide-fade-in .5s;
  }
  .slide-fade-leave-active {
    animation: slide-fade-out .5s;
  }
  @keyframes slide-fade-in {
    0% {
      transform: translateX(-50px);
      opacity: 0;
    }
    50% {
      transform: translateX(50px);
      opacity: 0.5;
    }
    100% {
      transform: translateX(0);
      opacity: 1;
    }
  }
  @keyframes slide-fade-out {
    0% {
      transform: translateX(0);
      opacity: 1;
    }
    50% {
      transform: translateX(50px);
      opacity: 0.5;
    }
    100% {
      transform: translateX(-50px);
      opacity: 0;
    }
  }
</style>
