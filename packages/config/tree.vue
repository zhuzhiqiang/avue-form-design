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
    <el-form-item label="字典配置"><br>
      <el-tabs v-model="dicOption"
               stretch
               @tab-click="handleTabClick">
        <el-tab-pane label="静态数据"
                     name="1">
          <el-tree ref="tree"
                   :data="data.dicData"
                   default-expand-all
                   draggable
                   node-key="value"
                   :expand-on-click-node="false">
            <span class="custom-tree-node"
                  slot-scope="{ node, data }">
              <span>{{ node.label }}</span>
              <span>
                <el-button type="text"
                           size="mini"
                           icon="el-icon-plus"
                           @click="handleNodeAdd(data)"></el-button>
                <!--                <el-button class="warning" type="text" size="mini" icon="el-icon-edit"-->
                <!--                           @click="handleNodeEdit(data)"></el-button>-->
                <el-button class="danger"
                           type="text"
                           size="mini"
                           icon="el-icon-delete"
                           @click="handleNodeRemove(node, data)"></el-button>
              </span>
            </span>
          </el-tree>
          <div style="margin-left: 22px;">
            <el-button type="text"
                       @click="handleParentNodeAdd">添加父级
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
    <el-form-item label="当有子级时,是否可选择父级">
      <el-switch v-model="data.parent"></el-switch>
    </el-form-item>
    <el-form-item label="是否多选">
      <el-switch v-model="data.multiple"></el-switch>
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

    <el-dialog :visible.sync="dialogVisible"
               :rules="dialogRules"
               :before-close="beforeClose">
      <el-form ref="dialogForm"
               :model="dialogForm"
               label-width="80px">
        <el-form-item label="label">
          <el-input v-model="dialogForm.label"
                    placeholder="label"></el-input>
        </el-form-item>
        <el-form-item label="value">
          <el-input v-model="dialogForm.value"
                    placeholder="value"
                    :type="this.dialogInputType">
            <el-select v-model="dialogInputType"
                       slot="append"
                       placeholder="数据类型"
                       style="width: 100px">
              <el-option label="String"
                         value="text"></el-option>
              <el-option label="Number"
                         value="number"></el-option>
            </el-select>
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="handleDialogAdd"
                   v-if="dialogStatus == 'add'">确 定</el-button>
        <!--        <el-button type="primary" @click="handleDialogUpdate" v-else>确 定</el-button>-->
      </span>
    </el-dialog>
  </div>
</template>
<script>


export default {
  name: "config-tree",
  props: ['data'],
  data () {
    return {
      validator: {
        type: null,
        required: null,
        pattern: null,
        length: null
      },
      dicOption: '1',
      dicCopy: this.deepClone(this.data.dicData),
      dialogForm: {},
      dialogVisible: false,
      dialogRules: {
        label: { required: true, message: '请输入label' },
        value: { required: true, message: '请输入value' },
      },
      dialogStatus: 'add',
      selectData: undefined,
      dialogInputType: 'text'
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
    },
    handleParentNodeAdd () {
      this.selectData = undefined
      this.dialogStatus = 'add';
      this.dialogVisible = true;
    },
    handleNodeAdd (data) {
      this.selectData = data;
      this.dialogStatus = 'add';
      this.dialogVisible = true;
    },
    handleNodeRemove (node, data) {
      const parent = node.parent;
      const children = parent.data.children || parent.data;
      const index = children.findIndex(d => d.id === data.id);
      children.splice(index, 1);
    },
    handleDialogAdd () {
      this.$refs.dialogForm.validate((valid) => {
        if (valid) {
          const { label, value } = this.dialogForm;
          const node = this.$refs.tree.getNode(value)
          if (node) this.$message.error("value重复")
          else {
            const data = this.selectData
            const newNode = {
              label,
              value: this.dialogInputType == 'number' ? new Number(value) : value,
            }
            if (data) {
              if (!data.children) this.$set(data, 'children', [])
              data.children.push(newNode)
            } else {
              this.$set(this.data.dicData, this.data.dicData.length, newNode)
            }
            this.beforeClose()
          }
        }
      })
    },
    beforeClose () {
      this.$refs.dialogForm.clearValidate()
      this.dialogForm = {}
      this.dialogVisible = false
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
    },
    'data.multiple': function (val) {
      if (val) this.data.defaultValue = []
      else delete this.data.defaultValue
    }
  }
}
</script>
<style lang="scss" scoped>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
