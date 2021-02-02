<template>
  <div>
    <!--    面包屑导航-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <el-steps
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="已完成"></el-step>
      </el-steps>
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          :tab-position="'left'"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabclicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="name" prop="goods_name">
              <el-input v-model="addForm.goods_name"> </el-input>
            </el-form-item>
            <el-form-item label="price" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"> </el-input>
            </el-form-item>
            <el-form-item label="weight" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"> </el-input>
            </el-form-item>
            <el-form-item label="amount" prop="goods_number">
              <el-input v-model="addForm.goods_number"> </el-input>
            </el-form-item>
            <el-form-item label="商品分类">
              <!--          options:数据源-->
              <!--          props:指定配置对象-->
              <div class="block">
                <el-cascader
                  expand-trigger="hover"
                  :options="catelist"
                  :props="cateprops"
                  v-model="addForm.goods_cat"
                  @change="handleChange"
                  clearable
                >
                </el-cascader>
              </div>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  v-for="(cb, idx) in item.attr_vals"
                  :key="idx"
                  :label="cb"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"> </el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!--            action:api地址-->
            <el-upload
              :action="uploadUrl"
              :headers="headersObj"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!--            富文本编辑器-->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button type="primary" class="add" @click="add"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <el-dialog title="预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'loadsh'
export default {
  data() {
    return {
      activeIndex: '0',
      addForm: {
        // 添加表单对象
        goods_name: '',
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [{ required: true, message: '商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '价格', trigger: 'blur' }],
        goods_weight: [{ required: true, message: 'weight', trigger: 'blur' }],
        goods_number: [{ required: true, message: 'number', trigger: 'blur' }],
        goods_cat: [{ required: true, message: 'cat', trigger: 'blur' }]
      },
      catelist: [],
      cateprops: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manyTableData: [],
      onlyTableData: [],
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewVisible: false
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        this.$message.error('获取商品失败')
      } else {
        this.catelist = res.data
        // console.log('this.catelist :>> ', this.catelist)
      }
    },
    handleChange() {
      // console.log(this.addForm.goods_cat)
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabclicked() {
      // console.log('click')
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' }
          }
        )
        // console.log(res)
        if (res.meta.status === 200) {
          res.data.forEach(item => {
            item.attr_vals =
              item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
          })
          this.manyTableData = res.data
        }
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' }
          }
        )
        if (res.meta.status === 200) {
          this.onlyTableData = res.data
        }
      }
    },
    // 处理图片预览
    handlePreview(file) {
      // console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理图片移除
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(x => x.pic === filePath)
      this.addForm.pics.splice(i, 1)
      // console.log(this.addForm)
    },
    // 图片上传成功时的钩子
    handleSuccess(res) {
      // console.log(res)
      // 1.拼接图片信息
      // 2.push到pics[]
      const picInfo = { pic: res.data.tmp_path }
      this.addForm.pics.push(picInfo)
      // console.log(this.addForm)
    },
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          // 执行添加
          // loadsh深拷贝
          const form = _.cloneDeep(this.addForm)
          form.goods_cat = form.goods_cat.join(',')
          // 处理静态属性
          this.manyTableData.forEach(item => {
            const newInfo = {
              attr_id: item.attr_id,
              attr_value: item.attr_vals.join(',')
            }
            this.addForm.attrs.push(newInfo)
          })
          this.onlyTableData.forEach(item => {
            const newInfo = {
              attr_id: item.attr_id,
              attr_value: item.attr_vals
            }
            this.addForm.attrs.push(newInfo)
          })
          form.attrs = this.addForm.attrs
          console.log(form)
          // 发起请求
          const { data: res } = await this.$http.post('goods', form)
          if (res.meta.status !== 201) {
            this.$message.error('添加失败')
          } else {
            this.$message.success('成功')
            this.$router.push('/goods')
          }
        } else {
          this.$message.error('请添加必要表单项')
        }
      })
    }
  },
  created() {
    this.getCateList()
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
}
.el-step__title {
  font-size: 13px;
}
.el-checkbox {
  margin: 0 5px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.add {
  margin-top: 15px;
}
</style>
