<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col span="4">
          <el-button type="primary">添加</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe style="width: 100%">
        <el-table-column type="expand" width="80px">
          <template slot-scope="scope">
            <el-row v-for="(item,index) in scope.row.children"
                    :key="item.id" :class="['btomborder',index===0?'topborder':'','aligncenter']">
              <el-col span="5">
                <el-tag>{{ item.authName }}</el-tag>
                <i class="el-icon-caret-right"></i></el-col>
              <el-col span="19">
                <el-row v-for="(item2,index2) in item.children" :key="item2.id"
                        :class="[index2!==0?'topborder':'','aligncenter']">
                  <el-col span="6">
                    <el-tag type="success">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i></el-col>
                  <el-col span="13">
                    <el-tag v-for="item3 in item2.children" :key="item3.id" closable type="warning"
                            @close="confirmDeleteRolesRight(scope.row,item3.id)">{{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" width="80px"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button icon="el-icon-edit" type="primary">编辑</el-button>
            <el-button icon="el-icon-delete" type="danger">删除</el-button>
            <el-button icon="el-icon-setting" type="warning" @click="DistributRight(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog  :visible.sync="isDistributeRightDialog" title="提示" width="30%" @close="closeDialogOfDistribute">
      <el-tree ref="treeRef" :data="rightlist" :props="distributDafaultProps" show-checkbox default-expand-all node-key="id" :default-checked-keys="defaultCheckKeys"></el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="isDistributeRightDialog = false">取 消</el-button>
    <el-button type="primary" @click="reDistributeInDialog">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      isDistributeRightDialog: false,
      rightlist: [],
      distributDafaultProps: {
        children: 'children',
        label: 'authName'
      },
      defaultCheckKeys: [],
      currentTableRowID: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('fail to get roles list')
      }
      this.rolesList = res.data
      // console.log(this.rolesList)
    },
    confirmDeleteRolesRight (val, id) {
      this.$confirm('准备删除此项权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${val.id}/rights/${id}`)
        if (res.meta.status !== 200) {
          return this.$message.error('fail to delete,maybe networks mistakes')
        }
        val.children = res.data
        this.$message.success('删除成功')
      }, () => {
        this.$message.info('cancel this time of delete')
      }
      )
    },
    async DistributRight (val) {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('faling to get DIstribute rights')
      }
      this.rightlist = res.data
      // console.log(this.rightlist)
      this.labelTreeArray(val, this.defaultCheckKeys)
      console.log(this.defaultCheckKeys)
      this.currentTableRowID = val.id
      this.isDistributeRightDialog = true
    },
    labelTreeArray (val, arr) {
      if (val.children) {
        for (const x of val.children) {
          this.labelTreeArray(x, arr)
        }
      } else {
        arr.push(val.id)
      }
    },
    closeDialogOfDistribute () {
      this.defaultCheckKeys = []
    },
    async reDistributeInDialog () {
      const keyarr = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const rid = keyarr.join(',')
      const { data: res } = await this.$http.post(`roles/${this.currentTableRowID}/rights`, { rids: rid })
      if (res.meta.status !== 200) {
        return this.$message.error('fail in reDistributeInDialog')
      }
      this.$message.success('success in reDistributeInDialog')
      this.getRolesList()
      this.isDistributeRightDialog = false
    }
  }
}
</script>

<style lang="less" scoped>
.btomborder {
  border-bottom: 1px solid #eee;
}

.topborder {
  border-top: 1px solid #eee;
}

.el-tag {
  margin: 7px;
}

.aligncenter {
  display: flex;
  align-items: center;
}
</style>
