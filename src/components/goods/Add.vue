<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert
        title="添加商品信息"
        :closable="false"
        type="info"
        center
        show-icon
      >
      </el-alert>
      <el-steps
        align-center
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          :before-leave="beforeTableave"
          v-model="activeIndex"
          :tab-position="'left'"
          @tab-click="tabClickd"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input type="number" v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input type="number" v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input type="number" v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="{
                  expandTrigger: 'hover',
                  label: 'cat_name',
                  value: 'cat_id',
                  children: 'children',
                }"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单的item项 -->
            <el-form-item
              v-for="item in manyTableDate"
              :key="item.attr_id"
              :label="item.attr_name"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                  :label="cb"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableDate"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="uploadUrl"
              :headers="headerObj"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button @click="add" class="btnAdd" type="primary"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img class="previewImg" :src="previewPath" alt="" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: [],
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' },
        ],
      },
      cateList: [],
      manyTableDate: [],
      onlyTableDate: [],
      uploadUrl: 'http://timemeetyou.com:8889/api/private/v1/upload',
      headerObj: {
        Authorization: window.sessionStorage.getItem('token'),
      },
      previewPath: '',
      previewVisible: false,
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取所有商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
      console.log(res.data)
    },
    // 级联选择器选中项变化
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforeTableave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabClickd() {
      // 访问的是动态参数
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' },
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableDate = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' },
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.onlyTableDate = res.data
      }
    },
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex((x) => {
        x.pic === filePath
      })
      this.addForm.pics.splice(i, 1)
    },
    // 监听图片长传成功
    handleSuccess(res) {
      const picInfo = { pic: res.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    add() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        // lodash 深拷贝
        const from = _.cloneDeep(this.addForm)
        from.goods_cat = from.goods_cat.join(',')
        // 动态参数和静态属性
        this.manyTableDate.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' '),
          }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyTableDate.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals,
          }
          this.addForm.attrs.push(newInfo)
        })
        from.attrs = this.addForm.attrs
        const { data: res } = await this.$http.post('goods',from)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        this.$router.push('/goods')
      })
    },
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      } else {
        return null
      }
    },
  },
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>