<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col span="4">
          <el-button type="primary" @click="addGoodsItems">添加</el-button>
        </el-col>
      </el-row>
      <tree-table :columns="columns" :data="cateList" :expand-type="false" :selection-type="false" border index-text="#"
                  show-index>
        <template slot="isvalidate" scope="scope">
          <i v-if="!scope.row.cat_deleted" class="el-icon-success" style="color:lightgreen"></i> <i v-else
                                                                                                    class="el-icon-error"
                                                                                                    style="color:red"></i>
        </template>
        <template slot="slot_level" scope="scope">
          <el-tag v-if="scope.row.cat_level==0" type="success">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level==1" type="warning">二级</el-tag>
          <el-tag v-else type="danger">三级</el-tag>
        </template>
        <template slot="opt">
          <el-button icon="el-icon-edit" type="primary">编辑</el-button>
          <el-button icon="el-icon-edit" type="danger">删除</el-button>
        </template>
      </tree-table>
      <el-pagination :current-page="queryInfo.pagenum" :page-size="queryInfo.pagesize" :page-sizes="[2, 5, 10, 20]"
                     :total="total" layout="total, sizes, prev, pager, next, jumper"
                     @size-change="handleSizeChange"
                     @current-change="handleCurrentChange">
      </el-pagination>
    </el-card>
    <!--    添加的dialog-->
    <el-dialog
      :visible.sync="isdialogAddGoods"
      title="添加商品分类"
      width="30%" @close="colseAddDialog">
      <el-form ref="addFormRef" :model="addForm" :rules="rulesForm" label-width="100px">
        <el-form-item label="商品名称" prop="cat_name">
          <el-input v-model="addForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="商品分类">
          <el-cascader
            v-model="parentvalue"
            :options="parentCateList"
            :props="cascadeProps" clearable
            @change="parentCateChange">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="isdialogAddGoods = false">取 消</el-button>
    <el-button type="primary" @click="addCateForm">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      total: '',
      columns: [{
        label: '商品类别',
        prop: 'cat_name'
      }, {
        label: '是否有效',
        type: 'template',
        template: 'isvalidate'
      }, {
        label: '排序',
        type: 'template',
        template: 'slot_level'
      }, {
        label: '操作',
        type: 'template',
        template: 'opt'
      }],
      isdialogAddGoods: false,
      addForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      rulesForm: {
        cat_name: [
          {
            required: true,
            message: '请输入名称!!!',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 5 个字符',
            trigger: 'blur'
          }
        ]
      },
      parentCateList: [],
      cascadeProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      parentvalue: []

    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('fail to get cate list')
      }
      this.cateList = res.data.result
      this.total = res.data.total
      // console.log((res))
    },
    handleSizeChange (newsize) {
      this.queryInfo.pagesize = newsize
      this.getCateList()
    },
    handleCurrentChange (newpage) {
      this.queryInfo.pagenum = newpage
      this.getCateList()
    },
    addGoodsItems () {
      this.getParentCateList()
      this.isdialogAddGoods = true
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('fail in getParentCateList')
      }
      this.parentCateList = res.data
      console.log(res)
    },
    parentCateChange () {
      if (this.parentvalue.length !== 0) {
        this.addForm.cat_level = this.parentvalue.length
        this.addForm.cat_pid = this.parentvalue[this.parentvalue.length - 1]
      } else {
        this.addForm.cat_level = 0
        this.addForm.cat_pid = 0
      }
      // console.log(this.parentvalue)
    },
    async addCateForm () {
      const { data: res } = await this.$http.post('categories', this.addForm)
      if (res.meta.status !== 201) {
        return this.$message.error('fail in addCateForm')
      }
      this.getCateList()
      this.isdialogAddGoods = false
      // console.log(this.addForm)
    },
    colseAddDialog () {
      console.log('adadad')
      this.$refs.addFormRef.resetFields()
      this.parentvalue = []
      this.addForm.cat_level = 0
      this.addForm.cat_pid = 0
    }
  }
}
</script>

<style lang="less" scoped>
.el-row {
  margin-bottom: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
