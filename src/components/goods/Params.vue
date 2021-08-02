<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert
        show-icon
        :closable="false"
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
      >
      </el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="{
              expandTrigger: 'hover',
              value: 'cat_id',
              label: 'cat_name',
              children: 'children',
            }"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            @click="addDialogVisible = true"
            :disabled="isBtnDisabled"
            size="mini"
            type="primary"
            >添加参数</el-button
          >
          <!-- 动态参数的表格 -->
          <el-table :data="manyTabelData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  @close="handleClose(i, scope.row)"
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  @click="showEditDialog(scope.row.attr_id)"
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  >编辑</el-button
                >
                <el-button
                  @click="removeParams(scope.row.attr_id)"
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            @click="addDialogVisible = true"
            :disabled="isBtnDisabled"
            size="mini"
            type="primary"
            >添加属性</el-button
          >
          <el-table :data="onlyTabelData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  @close="handleClose(i, scope.row)"
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="属性名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  @click="showEditDialog(scope.row.attr_id)"
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  >编辑</el-button
                >
                <el-button
                  @click="removeParams(scope.row.attr_id)"
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormref"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormref"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类数据
      cateList: [],
      //   级联选择款选中数据
      selectedCateKeys: [],
      //   被激活的页签名称
      activeName: 'many',
      // 动态参数的数据
      manyTabelData: [],
      // 静态属性的数据
      onlyTabelData: [],
      addDialogVisible: false,
      //添加参数数据
      addForm: {
        attr_name: '',
      },
      // 添加表单的数据规则
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' },
        ],
      },
      editDialogVisible: false,
      editForm: {
        attr_name: '',
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' },
        ],
      },
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
    },
    // 级联选择框变化会触发
    handleChange() {
      this.getParamsData()
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      this.getParamsData()
    },
    async getParamsData() {
      // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTabelData = []
        this.onlyTabelData = []
        return
      }
      //   获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        this.$message.error(res.meta.msg)
      }
      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示隐藏
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTabelData = res.data
      } else {
        this.onlyTabelData = res.data
      }
    },
    // 监听对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormref.resetFields()
    },
    // 确定按钮
    addParams() {
      this.$refs.addFormref.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName,
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击按钮展示修改对话框
    async showEditDialog(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed() {
      this.$refs.editFormref.resetFields()
    },
    editParams() {
      this.$refs.editFormref.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName,
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        this.editDialogVisible = false
        this.getParamsData()
      })
    },
    // 根据ID删除
    async removeParams(attr_id) {
      const str = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => err)
      // 用户取消了删除的操作
      if (str !== 'confirm') {
        return this.$message.info('已取消操作')
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getParamsData()
    },
    // 文本框失去焦点或按下确定
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' '),
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
    },
    showInput(row) {
      row.inputVisible = true
      // 让文本框获得焦点
      // nextTick 当页面上元素被重新渲染后才会执行指定的函数
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    },
  },
  computed: {
    //计算属性
    //   按钮禁用
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    // 当前选中的三级分类id
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      } else {
        return null
      }
    },
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    },
  },
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin-top: 15px;
}
.el-tag {
  margin-right: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>