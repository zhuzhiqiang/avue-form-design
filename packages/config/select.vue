<template>
  <div>
    <el-form-item label="占位内容">
      <el-input v-model="data.placeholder"
                placeholder="占位内容"></el-input>
    </el-form-item>
    <el-form-item label="默认值">
      <el-input v-model="data.valueDefault"
                placeholder="默认值"></el-input>
    </el-form-item>
    <el-form-item label="是否多选">
      <el-switch v-model="data.multiple"></el-switch>
    </el-form-item>
    <el-form-item label="是否可拖拽（需引入sortable.js）"
                  v-if="data.multiple">
      <el-switch v-model="data.drag"></el-switch>
    </el-form-item>
    <el-form-item label="多选数量限制"
                  v-if="data.multiple">
      <el-input-number v-model="data.limit"
                       controls-position="right"
                       placeholder="多选限制"
                       :min="0"
                       :max="data.dicData.length"></el-input-number>
    </el-form-item>
    <el-form-item label="字典配置"><br>
      <el-tabs v-model="dicOption"
               stretch
               @tab-click="handleTabClick">
        <el-tab-pane label="静态数据"
                     name="1">
          <draggable tag="ul"
                     :list="data.dicData"
                     :group="{ name: 'dic' }"
                     ghost-class="ghost"
                     handle=".drag-item">
            <li v-for="(item, index) in data.dicData"
                :key="index">
              <i class="drag-item el-icon-s-operation"
                 style="font-size: 16px; margin: 0 5px; cursor: move;"></i>
              <el-input style="margin-right: 5px;"
                        size="mini"
                        v-model="item.label"
                        placeholder="label"></el-input>
              <el-input size="mini"
                        v-model="item.value"
                        placeholder="value"></el-input>
              <el-button @click="handleRemoveFields(index)"
                         circle
                         plain
                         type="danger"
                         size="mini"
                         icon="el-icon-minus"
                         style="padding: 4px;margin-left: 5px;"></el-button>
            </li>
          </draggable>
          <div style="margin-left: 22px;">
            <el-button type="text"
                       @click="handleAddFields">添加列
            </el-button>
          </div>
        </el-tab-pane>
        <el-tab-pane label="远端数据"
                     name="2">
          网址
          <el-input v-model="data.dicUrl"
                    placeholder="远端数据字典网址"></el-input>
          请求方法
          <el-select v-model="data.dicMethod"
                     placeholder="请求方法"
                     style="width: 100%;">
            <el-option label="POST"
                       value="post"></el-option>
            <el-option label="GET"
                       value="get"></el-option>
          </el-select>
        </el-tab-pane>
      </el-tabs>
    </el-form-item>
    <el-form-item label="尺寸">
      <el-radio-group v-model="data.size"
                      size="mini">
        <el-radio-button label="medium">正常</el-radio-button>
        <el-radio-button label="small">小</el-radio-button>
        <el-radio-button label="mini">超小</el-radio-button>
      </el-radio-group>
    </el-form-item>
    <el-form-item label="是否可清空">
      <el-switch v-model="data.clearable"></el-switch>
    </el-form-item>
    <el-form-item label="是否可搜索">
      <el-switch v-model="data.filterable"></el-switch>
    </el-form-item>
    <el-form-item label="是否禁用">
      <el-switch v-model="data.disabled"></el-switch>
    </el-form-item>
    <el-form-item label="是否可见">
      <el-switch v-model="data.display"></el-switch>
    </el-form-item>
    <el-form-item label="是否必填">
      <el-switch v-model="data.required"></el-switch>
    </el-form-item>
  </div>
</template>
<script>
import Draggable from 'vuedraggable'


export default {
  name: "config-select",
  props: ['data'],
  components: { Draggable },
  data () {
    return {
      validator: {
        type: null,
        required: null,
        pattern: null,
        length: null
      },
      dicOption: '1',
      dicCopy: this.deepClone(this.data.dicData)
    }
  },
  methods: {
    generateRule () {
      const rules = [];
      Object.keys(this.validator).forEach(key => {
        if (this.validator[key]) rules.push(this.validator[key])
      })
      this.data.rules = rules
    },
    handleRemoveFields (index) {
      this.data.dicData.splice(index, 1)
    },
    handleAddFields () {
      const i = Math.ceil(Math.random() * 99999)
      this.data.dicData.push({ label: `字段${i}`, value: `col_${i}` })
    },
    handleTabClick (tab) {
      const { name } = tab;
      if (name == '1') {
        delete this.data.dicUrl
        delete this.data.dicMethod
        if (this.data.dicData.length == 0)
          this.data.dicData = this.dicCopy
      } else {
        this.data.dicData = []
        this.data.dicUrl = ''
      }
    }
  },
  watch: {
    'data.required': function (val) {
      if (val) this.validator.required = { required: true, message: `请选择${this.data.label}` }
      else this.validator.required = null

      this.generateRule()
    },
    'data.dicData': {
      handler (val) {
        if (val && val.length > 0 && !Object.is(val, this.dicCopy)) this.dicCopy = this.deepClone(val)
      },
      deep: true
    }
  }
}
</script>
