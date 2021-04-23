<template>
  <div class="UserList">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--  card part-->
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8">
          <!--        一行总共24个span，由这些列分享-->
          <el-input v-model="queryInfo.query" clearable placeholder="请输入内容" @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="8">
          <el-button type="primary" @click="isShowDialog=true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="姓名"
          prop="username">
        </el-table-column>
        <el-table-column
          label="邮箱"
          prop="email">
        </el-table-column>
        <el-table-column
          label="电话"
          prop="mobile">
        </el-table-column>
        <el-table-column
          label="角色"
          prop="role_name">
        </el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button circle icon="el-icon-edit" type="primary" @click="changeUserInfo(scope.row.id)"></el-button>
            <el-button circle icon="el-icon-delete" type="danger" @click="confirmDelete(scope.row.id)"></el-button>
            <el-tooltip :enterable="false" content="设置" effect="dark" placement="top">
              <el-button circle icon="el-icon-setting" type="warning"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!--      partition page-->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-size="queryInfo.pagesize"
        :page-sizes="[1, 2, 5, 10]"
        :total="total"
        layout="total, sizes, prev, pager, next, jumper"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"></el-pagination>
    </el-card>
    <!--    添加的dialog-->
    <el-dialog
      :visible.sync="isShowDialog"
      title="提示"
      width="30%" @close="clearFormBeforeClose">
      <el-form ref="inputFormRef" :model="inputForm" :rules="inputFormRules" label-width="100px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="inputForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="inputForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="inputForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="inputForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="isShowDialog= false">取 消</el-button>
    <el-button type="primary" @click="addFormItem">确 定</el-button>
  </span>
    </el-dialog>
    <!-- 修改的dialog-->
    <el-dialog :visible.sync="isAddDialog"
               title="提示"
               width="30%"
               @close="clearChangingForm">
      <el-form ref="changeFormRef" :model="changingForm" :rules="changingFormRules" label-width="100px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="changingForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="changingForm.mobile"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="changingForm.email"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="isAddDialog= false">取 消</el-button>
    <el-button type="primary" @click="changeDialogForm">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    var checkphone = (rule, value, callback) => {
      const reg = /^1\d{6,15}$/
      if (reg.test(value)) {
        callback()
      } else {
        callback(new Error('wrong number'))
      }
    }
    var checkemail = (rule, value, callback) => {
      const reg = /([a-z]|[A-Z]|[0-9])+@([a-z]|[A-Z])+\.com$/
      if (reg.test(value)) {
        callback()
      } else {
        callback(new Error('wrong emal'))
      }
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      isShowDialog: false,
      inputForm: {
        username: '',
        email: '',
        mobile: '',
        password: ''
      },
      isAddDialog: false,
      inputFormRules: {
        username: [{
          required: true,
          message: '请填写用户名',
          trigger: 'blur'
        }, {
          min: 5,
          max: 15,
          message: '用户名必须是5-15位',
          trigger: 'blur'
        }],
        password: [{
          required: true,
          message: '请填写密码',
          trigger: 'blur'
        }, {
          min: 5,
          max: 15,
          message: '密码必须是5-15位',
          trigger: 'blur'
        }],
        email: [{
          required: true,
          message: '请填写邮箱',
          trigger: 'blur'
        }, {
          min: 10,
          max: 30,
          message: '邮箱必须是10-30位',
          trigger: 'blur'
        }, {
          validator: checkemail,
          trigger: 'blur'
        }],
        mobile: [{
          required: true,
          message: '请填写手机号1',
          trigger: 'blur'
        }, {
          min: 5,
          max: 15,
          message: '手机号必须是5-15位',
          trigger: 'blur'
        }, {
          validator: checkphone,
          trigger: 'blur'
        }]
      },
      changingForm: {},
      changingFormRules: {
        password: [{
          required: true,
          message: '请填写密码',
          trigger: 'blur'
        }, {
          min: 5,
          max: 15,
          message: '密码必须是5-15位',
          trigger: 'blur'
        }],
        email: [{
          required: true,
          message: '请填写邮箱',
          trigger: 'blur'
        }, {
          min: 10,
          max: 30,
          message: '邮箱必须是10-30位',
          trigger: 'blur'
        }, {
          validator: checkemail,
          trigger: 'blur'
        }],
        mobile: [{
          required: true,
          message: '请填写手机号1',
          trigger: 'blur'
        }, {
          min: 5,
          max: 15,
          message: '手机号必须是5-15位',
          trigger: 'blur'
        }, {
          validator: checkphone,
          trigger: 'blur'
        }]
      }
    }
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        this.$message.error('failing to lode user list')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange (newsize) {
      this.queryInfo.pagesize = newsize
      this.getUserList()
    },
    handleCurrentChange (newpage) {
      this.queryInfo.pagenum = newpage
      this.getUserList()
    },
    async userStateChange (wwws) {
      // console.log('ddddddddddd', wwws)
      const { data: res } = await this.$http.put(`users/${wwws.id}/state/${wwws.mg_state}`)
      if (res.meta.status !== 200) {
        wwws.mg_state = !wwws.mg_state
        return this.$message.error('fail to change the status')
      }
      return this.$message.success('success to change the status')
    },
    clearFormBeforeClose () {
      console.log('has closed!!!!!!!!!!!!!!!')
      this.$refs.inputFormRef.resetFields()
    },
    addFormItem () {
      this.$refs.inputFormRef.validate(async (valid) => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post('/users', this.inputForm)
        if (res.meta.status !== 201) {
          return this.$message.error('fail to add members')
        } else {
          this.$message.success('success')
          this.getUserList()
          this.isShowDialog = false
        }
      })
    },
    async changeUserInfo (id) {
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('fail to search')
      }
      this.changingForm = res.data
      this.isAddDialog = true
    },
    clearChangingForm () {
      this.$refs.changeFormRef.resetFields()
    },
    changeDialogForm () {
      this.$refs.changeFormRef.validate(async (valid) => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.put(`users/${this.changingForm.id}`, {
          email: this.changingForm.email,
          mobile: this.changingForm.mobile
        })
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('fail to changeing user info')
        }
        this.$message.success('successful to change the user info')
        this.isAddDialog = false
        this.getUserList()
      })
    },
    confirmDelete (id) {
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`users/${id}`)
        if (res.meta.status === 200) {
          this.$message.success('success to delete user info')
          this.getUserList()
        } else {
          this.$message.error('fail to delete users,maybe networks')
        }
      }, () => {
        this.$message.info('cancel to delete')
      }
      )
    }
  },
  created () {
    this.getUserList()
  }
}
</script>

<style lang="less" scoped>
.el-row {
  margin-bottom: 15px;
}
</style>
