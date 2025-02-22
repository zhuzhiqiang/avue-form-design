<template>
  <div class="widget-form-container">
    <el-form :label-position="data.labelPosition || 'left'"
             :label-width="data.labelWidth ? `${data.labelWidth}px` : '100px' "
             :label-suffix="data.labelSuffix"
             :model="form"
             ref="widgetForm">
      <el-row :gutter="data.gutter">
        <draggable class="widget-form-list"
                   :list="data.column"
                   :group="{ name: 'form' }"
                   ghost-class="ghost"
                   :animation="300"
                   @add="handleWidgetAdd">
          <template v-for="(column, index) in data.column">
            <div class="widget-form-table"
                 v-if="column.type == 'dynamic'"
                 :key="index"
                 :class="{ active: selectWidget.prop == column.prop }"
                 @click="handleSelectWidget(index)">
              <widget-form-table :data="data"
                                 :column="column"
                                 :index="index"
                                 :select.sync="selectWidget"></widget-form-table>
            </div>
            <div class="widget-form-group"
                 v-else-if="column.type == 'group'"
                 :key="index"
                 :class="{ active: selectWidget.prop == column.prop }"
                 @click="handleSelectWidget(index)">
              <widget-form-group :data="data"
                                 :column="column"
                                 :index="index"
                                 :select.sync="selectWidget">
              </widget-form-group>
            </div>
            <el-col v-else
                    :key="index"
                    :md="column.span || 12"
                    :xs="24"
                    :offset="column.offset || 0">
              <el-form-item class="widget-form-item"
                            :label="column.label"
                            :prop="column.prop"
                            :class="{ active: selectWidget.prop == column.prop, 'required': column.required }"
                            @click.native="handleSelectWidget(index)">
                <widget-form-item :item="column"></widget-form-item>
                <el-button title="删除"
                           @click.stop="handleWidgetDelete(index)"
                           class="widget-action-delete"
                           v-if="selectWidget.prop == column.prop"
                           circle
                           plain
                           type="danger">
                  <i class="iconfont icon-delete"></i>
                </el-button>
                <el-button title="复制"
                           @click.stop="handleWidgetClone(index)"
                           class="widget-action-clone"
                           v-if="selectWidget.prop == column.prop"
                           circle
                           plain
                           type="primary">
                  <i class="iconfont icon-copy"></i>
                </el-button>
              </el-form-item>
            </el-col>
          </template>
        </draggable>
      </el-row>
    </el-form>
  </div>
</template>

<script>
import WidgetFormItem from './WidgetFormItem'
import WidgetFormTable from './WidgetFormTable'
import WidgetFormGroup from './WidgetFormGroup'
import Draggable from 'vuedraggable'

export default {
  name: 'widget-form',
  components: { Draggable, WidgetFormItem, WidgetFormTable, WidgetFormGroup },
  props: ['data', 'select'],
  data () {
    return {
      selectWidget: this.select,
    }
  },
  methods: {
    handleSelectWidget (index) {
      this.selectWidget = this.data.column[index]
    },
    handleWidgetAdd (evt) {
      const newIndex = evt.newIndex;
      const data = this.deepClone(this.data.column[newIndex]);
      if (!data.prop) data.prop = Date.now() + '_' + Math.ceil(Math.random() * 99999)
      delete data.icon
      delete data.subfield
      this.$set(this.data.column, newIndex, { ...data })
      this.handleSelectWidget(newIndex)
    },
    handleWidgetDelete (index) {
      if (this.data.column.length - 1 === index) {
        if (index === 0) this.selectWidget = {}
        else this.handleSelectWidget(index - 1)
      } else this.handleSelectWidget(index + 1)

      this.$nextTick(() => {
        this.data.column.splice(index, 1)
      })
    },
    handleWidgetClone (index) {
      let cloneData = this.deepClone(this.data.column[index])
      cloneData.prop = Date.now() + '_' + Math.ceil(Math.random() * 99999)
      this.data.column.splice(index, 0, cloneData)
      this.$nextTick(() => {
        this.handleSelectWidget(index + 1)
      })
    },
  },
  watch: {
    select (val) {
      this.selectWidget = val
    },
    selectWidget: {
      handler (val) {
        this.$emit('update:select', val)
      },
      deep: true
    }
  }
}
</script>
