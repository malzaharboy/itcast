<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--  卡片区-->
    <el-card>
      <el-row gutter="7">
        <el-col span="8">
          <el-input v-model="queryInfo.query" clearable placeholder="请输入内容" @clear="clearSearchInput">
            <el-button slot="append" icon="el-icon-search" @click="searchInput"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!--      table-->
      <el-table :data="orders" border stripe style="width: 100%">
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_id"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status=='1'" type="success">已付款</el-tag>
            <el-tag v-else type="danger">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send">
        </el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time|dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <!--          <template slot-scope="scope">-->
          <el-button circle icon="el-icon-edit" type="primary" @click="editInTable"></el-button>
          <el-button circle icon="el-icon-location" type="success" @click="showTimeline"></el-button>
          <!--          </template>-->
        </el-table-column>
      </el-table>
      <!--      partition-->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-size="queryInfo.pagesize"
        :page-sizes="[10, 15, 20, 30]"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="pageSizeChange"
        @current-change="pageCurrentChange">
      </el-pagination>
    </el-card>
    <!--    edit in table dialog-->
    <el-dialog
      :visible.sync="dialogVisibleEdit"
      title="修改地址" width="30%"
      @close="closeEditDialog">
      <el-form ref="ruleForm" :model="editAdressForm" :rules="editAdressFormRules" class="demo-ruleForm"
               label-width="100px">
        <el-form-item label="省市区/县" prop="selectCity">
          <el-cascader
            v-model="editAdressForm.selectCity"
            :options="citydata" :props="{ expandTrigger: 'hover' }"
            @change="cascadeChange"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="detailAdress">
          <el-input v-model="editAdressForm.detailAdress"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisibleEdit = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisibleEdit = false">确 定</el-button>
  </span>
    </el-dialog>
    <el-dialog :visible.sync="dialogVisibleLocation"
               title="地理位置" width="30%"
               @close="closeLocationDialog">
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in datatime"
          :key="index"
          :size="activity.size"
          :timestamp="activity.time">
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import citydata from '../../assets/citydata'

export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: '',
      citydata,
      orders: [],
      dialogVisibleEdit: false,
      editAdressForm: {
        selectCity: [],
        detailAdress: ''
      },
      editAdressFormRules: {
        selectCity: [{
          type: 'array',
          require: true,
          message: 'worng chose',
          trigger: 'blur'
        }],
        detailAdress: [{
          require: true,
          message: 'worng chose',
          trigger: 'blur'
        }]
      },
      datatime: [
        {
          time: '2018-05-10 09:39:00',
          ftime: '2018-05-10 09:39:00',
          context: '已签收,感谢使用顺丰,期待再次为您服务',
          location: ''
        },
        {
          time: '2018-05-10 08:23:00',
          ftime: '2018-05-10 08:23:00',
          context: '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
          location: ''
        },
        {
          time: '2018-05-10 07:32:00',
          ftime: '2018-05-10 07:32:00',
          context: '快件到达 [北京海淀育新小区营业点]',
          location: ''
        },
        {
          time: '2018-05-10 02:03:00',
          ftime: '2018-05-10 02:03:00',
          context: '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
          location: ''
        },
        {
          time: '2018-05-09 23:05:00',
          ftime: '2018-05-09 23:05:00',
          context: '快件到达 [北京顺义集散中心]',
          location: ''
        },
        {
          time: '2018-05-09 21:21:00',
          ftime: '2018-05-09 21:21:00',
          context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
          location: ''
        },
        {
          time: '2018-05-09 13:07:00',
          ftime: '2018-05-09 13:07:00',
          context: '顺丰速运 已收取快件',
          location: ''
        },
        {
          time: '2018-05-09 12:25:03',
          ftime: '2018-05-09 12:25:03',
          context: '卖家发货',
          location: ''
        },
        {
          time: '2018-05-09 12:22:24',
          ftime: '2018-05-09 12:22:24',
          context: '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
          location: ''
        },
        {
          time: '2018-05-08 21:36:04',
          ftime: '2018-05-08 21:36:04',
          context: '商品已经下单',
          location: ''
        }
      ],
      dialogVisibleLocation: false
    }
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('fail in getOrderList')
      }
      this.total = res.data.total
      this.orders = res.data.goods
    },
    pageSizeChange (x) {
      this.queryInfo.pagesize = x
      this.getOrderList()
    },
    pageCurrentChange (x) {
      this.queryInfo.pagenum = x
      this.getOrderList()
    },
    editInTable () {
      this.dialogVisibleEdit = true
    },
    cascadeChange () {
      console.log(this.editAdressForm.selectCity)
    },
    closeEditDialog () {
      this.editAdressForm.selectCity = []
      this.editAdressForm.detailAdress = ''
    },
    async showTimeline () {
      // const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      // console.log(res)
      this.dialogVisibleLocation = true
    },
    closeLocationDialog () {
    }
  },
  created () {
    this.getOrderList()
  }
}
</script>

<style scoped>
.el-row {
  margin-bottom: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
