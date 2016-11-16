<template>
   <table class="table is-bordered is-striped is-narrow" v-if="showTable">
            <thead>
              <tr>
                <th v-if="checkbox" class="table-checkbox"><input v-model="isAllChecked"  type="checkbox" @change="checkAll(isAllChecked)"/></th>
                <th v-for="column in columns" :colspan="column.key==='_actions'?actions.length:1">{{column.label}}</th>
              </tr>
            </thead>
            <tfoot v-if="havefoot">
              <tr>
                  <th v-if="checkbox" class="table-checkbox"><input v-model="isAllChecked" type="checkbox" @change="checkAll(isAllChecked)"/></th>
                 <th v-for="column in columns" :colspan="column.key==='_actions'?actions.length:1">{{column.label}}</th>
              </tr>
            </tfoot>
            <tbody>
              <tr v-for='(item,index) in items'>
                <td v-if="checkbox" class="table-checkbox"> <input type="checkbox" :value='index' v-model="checkedIndexes"/></td>
                <td v-for="column in noActionColumns(columns)">
                  {{item[column.key]|dictFilter(column.key,dict)}}
                </td>
                <td v-if="actions" v-for="action in actions" class="is-icon">
                    <a :class='action.class' @click='handler(action.name,index)'><i v-if="action.icon" :class='action.icon'></i>{{action.label}}</a>
                </td>
              </tr>
            </tbody>
          </table>
</template>

<script>
export default {
  props: {
    checked: {
      type: Array,
      default() {
        return []
      }
    },
    checkbox: {
      type: Boolean,
      default: false
    },
    havefoot: {
      type: Boolean,
      default: true
    },
    items: {
      type: Array
    },
    columns: {
      type: Array
    },
    actions: {
      type: Array
    },
    dict: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  data() {
    return {
      checkedIndexes: this.checked,
      isAllChecked: false
    }
  },
  methods: {
    noActionColumns: function(columns) {
      return columns.filter(function(column) {
        return column.key !== '_actions'
      })
    },
    handler: function(action, index) {
      this.$emit(action, index)
    },
    checkAll: function(isAllChecked) {
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
    showTable: function() {
      return this.items.length ? true : false
    }
  },
  filters: {
      dictFilter: function(value, key, dict) {
          if(!value)return''
          if(!dict.hasOwnProperty(key))return value
          else return dict[key][value]
      }
  },
  watch: {
    items: function() {
      this.checkedIndexes = this.checked
    },
    checkedIndexes: function() {
      this.isAllChecked=this.checkedIndexes.length===this.items.length?true:false
      this.$emit('checkedChange', this.checkedIndexes)
    }
  }
}
</script>

<style scoped>
  .table-checkbox {
    text-align: center;
    font-size: 18px;
    line-height: 100%;
  }
</style>
