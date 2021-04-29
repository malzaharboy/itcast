<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--  卡片区-->
    <el-card>
      <el-row gutter="7">
        <el-col span="8">
          <el-input v-model="queryInfo.query" clearable placeholder="请输入内容" @clear="clearSearchInput">
            <el-button slot="append" icon="el-icon-search" @click="searchInput"></el-button>
          </el-input>
        </el-col>
        <el-col span="4">
          <el-button type="primary" @click="goTotheAdd">添加商品</el-button>
        </el-col>
      </el-row>
      <el-table :data="curPageData" border stripe style="width: 100%">
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格" prop="goods_price" width="95px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="80px"></el-table-column>
        <el-table-column label="创建时间" width="140px">
          <template slot-scope="scope">
            {{ scope.row.add_time|dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button icon="el-icon-edit" size="small" type="primary"></el-button>
            <el-button icon="el-icon-delete" size="small" type="danger"
                       @click="deleteTableRow(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-size="queryInfo.pagesize"
        :page-sizes="[5, 10, 20, 25]"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="pageSizeChange"
        @current-change="pageCurrentChange">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: '5'
      },
      curPageData: [],
      total: ''
    }
  },
  methods: {
    async getGoodsListdsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('fail in getGoodsListdsList')
      }
      this.curPageData = res.data.goods
      this.total = res.data.total
    },
    pageSizeChange (val) {
      console.log(val)
      this.queryInfo.pagesize = val
      this.getGoodsListdsList()
    },
    pageCurrentChange (val) {
      console.log(val)
      this.queryInfo.pagenum = val
      this.getGoodsListdsList()
    },
    searchInput () {
      this.getGoodsListdsList()
    },
    clearSearchInput () {
      this.getGoodsListdsList()
    },
    deleteTableRow (id) {
      this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`goods/${id}`)
        if (res.meta.status !== 200) {
          return this.$message.error('the delete meets mistakes')
        }
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        this.getGoodsListdsList()
      }, () => {
        this.$message.info('cancel the delete')
      })
    },
    goTotheAdd () {
      this.$router.push('/goods/add')
    }
  },
  created () {
    this.getGoodsListdsList()
  }
}
</script>

<style scoped>

</style>
