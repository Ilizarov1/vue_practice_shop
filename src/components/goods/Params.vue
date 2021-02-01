<template>
  <div>
    <!--    面包屑导航-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--    卡片视图区域-->
    <el-card>
      <el-alert
        title="只允许第三极分类设置相关参数"
        type="warning"
        :closable="false"
      >
      </el-alert>
      <!--      选择商品-->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类: </span>
          <!--  选择商品级联框-->
          <el-cascader
            expand-trigger="hover"
            :options="catelist"
            v-model="selectedKeys"
            :props="cateProps"
            @change="handleChange"
          >
          </el-cascader>
        </el-col>
      </el-row>
      <!--      Tab页签区-->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            @click="adddialogVisible = true"
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            >添加参数</el-button
          >
          <!--          动态参数表格-->
          <el-table :data="manyTableData" border stripe>
            <!--            展开行-->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!--                循环渲染tag-->
                <el-tag v-for="(item, idx) in scope.row.attr_vals" :key="idx">
                  {{ item }}</el-tag
                >
                <!--                输入文本框-->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!--                添加按钮-->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <!--            索引列-->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="name" prop="attr_name"></el-table-column>
            <el-table-column label="opt">
              <template>
                <el-button type="primary" icon="el-icon-edit" size="mini"
                  >编辑</el-button
                >
                <el-button type="danger" icon="el-icon-delete" size="mini"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            @click="adddialogVisible = true"
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            >添加属性</el-button
          >
          <!--          静态属性表格-->
          <el-table :data="onlyTableData" border stripe>
            <!--            索引列-->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="name" prop="attr_name"></el-table-column>
            <el-table-column label="opt">
              <template>
                <el-button type="primary" icon="el-icon-edit" size="mini"
                  >编辑</el-button
                >
                <el-button type="danger" icon="el-icon-delete" size="mini"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="adddialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="adddialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      catelist: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedKeys: [],
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      adddialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [{ required: true, message: '参数名称', trigger: 'blur' }]
      },
      inputVisible: false,
      // 控制文本框切换显示
      inputValue: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status === 200) {
        this.catelist = res.data
        // console.log(this.catelist)
      }
    },
    handleChange() {
      // console.log(this.selectedKeys)
      this.getParamsData()
    },
    handleTabClick() {
      // tab页签
      this.getParamsData()
    },
    async getParamsData() {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
      } else {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: this.activeName
            }
          }
        )
        if (res.meta.status === 200) {
          res.data.forEach(item => {
            item.attr_vals = item.attr_vals
              ? (item.attr_vals = item.attr_vals.split(' '))
              : []
            // 控制文本框的显示与隐藏
            item.inputVisible = false
            item.inputValue = ''
          })
          console.log(res.data)
          if (this.activeName === 'many') {
            this.manyTableData = res.data
          } else {
            this.onlyTableData = res.data
          }
        }
      }
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post(
            `categories/${this.cateId}/attributes`,
            {
              attr_name: this.addForm.attr_name,
              attr_sel: this.activeName
            }
          )
          if (res.meta.status === 201) {
            this.$message.success('添加成功')
            this.adddialogVisible = false
            this.getParamsData()
          }
        }
      })
    },
    // 文本框失去焦点或回车触发
    handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
      } else {
        row.attr_vals.push(row.inputValue.trim())
        row.inputValue = ''
        row.inputVisible = false
      }
    },
    // 展示文本输入框
    showInput(row) {
      row.inputVisible = true
      row.inputValue = ''
      // $nextTick()页面重新渲染后调用回调函数
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    }
  },
  computed: {
    isBtnDisabled() {
      if (this.selectedKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    cateId() {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    titleText() {
      // 动态标题
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 5px 10px;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
