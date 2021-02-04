<template>
  <div>
    <!--    面包屑导航-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input>
            <el-button slot="append" icon="el-icon-search"></el-button
          ></el-input>
        </el-col>
      </el-row>

      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="number" prop="order_number"></el-table-column>
        <el-table-column label="price" prop="order_price"></el-table-column>
        <el-table-column label="status" prop="pay_status">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="send" prop="is_send"></el-table-column>
        <el-table-column label="time" prop="create_time">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="opt">
          <template>
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showBox"
            ></el-button>
            <el-button
              @click="showProgressBox"
              type="success"
              size="mini"
              icon="el-icon-location"
            >
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--修改地址-->
    <el-dialog
      @close="addressDialogClosed"
      title="修改地址"
      :visible.sync="addressVisible"
      width="50%"
    >
      <el-form
        ref="addressFormRef"
        :model="addressForm"
        :rules="addressFormRules"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            :options="citydata"
            v-model="addressForm.address1"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!--展示物流进度-->
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
      <span>这是一段信息</span>
    </el-dialog>
  </div>
</template>

<script>
import citydata from './citydata'
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderList: [],
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [{ required: true, message: '请输入', trigger: 'blur' }],
        address2: [{ required: true, message: '请输入', trigger: 'blur' }]
      },
      citydata,
      progressVisible: false
    }
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      console.log(res.data)
      if (res.meta.status === 200) {
        this.total = res.data.total
        this.orderList = res.data.goods
      }
    },
    // 展示修改对话框
    showBox() {
      this.addressVisible = true
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    showProgressBox() {
      // const { data: res } = await this.$http.get('kuaidi/70365716896101')
      // if (res.meta.status === 200) {
      //   this.progressInfo = res.data
      // }
      this.progressVisible = true
    }
  },
  created() {
    this.getOrderList()
  }
}
</script>

<style scoped>
.el-cascader {
  width: 100%;
}
</style>
