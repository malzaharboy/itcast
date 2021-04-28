<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col span="20">
          <el-alert :closable="false" show-icon title="错误提示的文案" type="warning"></el-alert>
        </el-col>
      </el-row>
      <span>选择商品分类：</span>
      <el-cascader
        v-model="selectedvalues"
        :options="goodsParamsList"
        :props="cascadeProps"
        expand-trigger="hover"
        @change="cascadeChangeSelect">
      </el-cascader>
      <el-tabs v-model="tabActiveName" @tab-click="tableClick">
        <!--        动态参数-->
        <el-tab-pane label="动态参数" name="many">
          <el-button :disabled="isAbleadd" type="primary" @click="addParamsDialog">添加参数</el-button>
          <el-table :data="manyData" style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" :disable-transitions="false" closable
                        @close="closeTag(scope.row,i)">
                  {{ item }}
                </el-tag>
                <el-input v-if="scope.row.inputVisible" ref="inputRef" v-model="scope.row.inputValue" class="zzwInout"
                          size="small"
                          @blur="handleInputConfirm(scope.row)" @keyup.enter.native="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column
              type="index">
            </el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button circle icon="el-icon-edit" type="primary"
                           @click="editParamsIntable(scope.row.attr_id)"></el-button>
                <el-button circle icon="el-icon-delete" type="danger"
                           @click="deleteParams(scope.row.attr_id)"></el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!--        静态参数-->
        <el-tab-pane label="静态属性" name="only">
          <el-button :disabled="isAbleadd" type="primary" @click="addParamsDialog">添加属性</el-button>
          <el-table :data="onlyData" style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" :disable-transitions="false" closable
                        @close="closeTag(scope.row,i)">
                  {{ item }}
                </el-tag>
                <el-input v-if="scope.row.inputVisible" ref="inputRef" v-model="scope.row.inputValue" class="zzwInout"
                          size="small"
                          @blur="handleInputConfirm(scope.row)" @keyup.enter.native="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column
              type="index">
            </el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button circle icon="el-icon-edit" type="primary"
                           @click="editParamsIntable(scope.row.attr_id)"></el-button>
                <el-button circle icon="el-icon-delete" type="danger"
                           @click="deleteParams(scope.row.attr_id)"></el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!--    添加的dialog-->
    <el-dialog :visible.sync="isAddPramasDialog" title="添加参数" width="40%" @close="closeAddDialog">
      <el-form ref="addFormRef" :model="addForm" :rules="addFormrules" label-width="120px">
        <el-form-item :label="dialogText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="isAddPramasDialog = false">取 消</el-button>
    <el-button type="primary" @click="addPFormInDia">确 定</el-button>
  </span>
    </el-dialog>
    <!--    修改的对话框-->
    <el-dialog :visible.sync="isEditPramasDialog" title="修改参数" width="40%" @close="closeeditDialog">
      <el-form ref="editFormRef" :model="editForm" :rules="editFormrules" label-width="120px">
        <el-form-item :label="editdialogText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="isEditPramasDialog = false">取 消</el-button>
    <el-button type="primary" @click="editPFormInDia">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      goodsParamsList: [],
      selectedvalues: [],
      cascadeProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      tabActiveName: 'many',
      manyData: [],
      onlyData: [],
      isAddPramasDialog: false,
      addForm: {
        attr_name: ''
      },
      addFormrules: {
        attr_name: [{
          required: true,
          message: '请输入名称',
          trigger: 'blur'
        }]
      },
      isEditPramasDialog: false,
      editForm: {
        attr_name: ''
      },
      editFormrules: {
        attr_name: [{
          required: true,
          message: '请输入名称',
          trigger: 'blur'
        }]
      },
      currentAttrId: '',
      inputVisible: false,
      inputValue: ''
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('fail in getGoodsList')
      }
      this.goodsParamsList = res.data
      // console.log(this.goodsParamsList)
    },
    cascadeChangeSelect () {
      if (this.selectedvalues.length < 3) {
        this.selectedvalues = []
      }
      this.getFirstOrSecondAttributr()
    },
    async getFirstOrSecondAttributr () {
      if (!this.paramId) {
        this.manyData = []
        this.onlyData = []
        return
      }
      const { data: res } = await this.$http.get(`categories/${this.paramId}/attributes`, {
        params: {
          sel: this.tabActiveName
        }
      })
      // console.log('this', res)
      if (res.meta.status !== 200) {
        return this.$message.error('fail in getFirstOrSecondAttributr')
      }
      res.data.forEach((item) => {
        if (item.attr_vals) {
          item.attr_vals = item.attr_vals.split(' ')
        } else {
          item.attr_vals = []
        }
        item.inputValue = ''
        item.inputVisible = false
      })
      // for (const x of res.data) {
      //   x.attr_name = x.attr_v.split(' ')
      // }
      // console.log('ssssssss', res.data)
      if (this.tabActiveName === 'many') {
        this.manyData = res.data
      } else {
        this.onlyData = res.data
      }
    },
    tableClick () {
      this.getFirstOrSecondAttributr()
    },
    addParamsDialog () {
      this.isAddPramasDialog = true
    },
    closeAddDialog () {
      this.$refs.addFormRef.resetFields()
    },
    async addPFormInDia () {
      this.$refs.addFormRef.validate(async (validat) => {
        if (!validat) {
          return
        }
        const { data: res } = await this.$http.post(`categories/${this.paramId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.tabActiveName
        })
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('fail in addPFormInDia')
        }
        this.getFirstOrSecondAttributr()
        this.$message.success('succss in add params')
        this.isAddPramasDialog = false
      })
    },
    async editParamsIntable (attrId) {
      this.currentAttrId = attrId
      const { data: res } = await this.$http.get(`categories/${this.paramId}/attributes/${attrId}`, {
        params: {
          attr_sel: this.tabActiveName
        }
      })
      // console.log(res, this.paramId, attrId)
      if (res.meta.status !== 200) {
        return this.$message.error('fail in editParamsIntable')
      } else {
        this.editForm.attr_name = res.data.attr_name
      }

      // console.log('onlydata', this.onlyData)
      this.isEditPramasDialog = true
    },
    closeeditDialog () {
      this.$refs.editFormRef.resetFields()
      this.isEditPramasDialog = false
    },
    editPFormInDia () {
      this.$refs.editFormRef.validate(async (validat) => {
        if (!validat) {
          return
        }
        const { data: res } = await this.$http.put(`categories/${this.paramId}/attributes/${this.currentAttrId}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.tabActiveName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('fail in editPFormInDia')
        } else {
          this.$message.success('success in edit ')
          this.isEditPramasDialog = false
          this.$refs.editFormRef.resetFields()
          this.getFirstOrSecondAttributr()
        }
      })
    },
    deleteParams (attrId) {
      this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`categories/${this.paramId}/attributes/${attrId}`)
        if (res.meta.status !== 200) {
          this.$message.error('fail in deleteParams-solve')
        }
        this.getFirstOrSecondAttributr()
        this.$message.success('删除成功了')
      }, () => {
        this.$message.error('取消删除操作')
      })
    },
    async handleInputConfirm (x) {
      if (x.inputValue.trim() === '') {
        x.inputValue = ''
      } else {
        // console.log('adadad', x.attr_vals)
        x.attr_vals.push(x.inputValue)
        this.commitAlreadyChangeTag(x)
      }
      x.inputValue = ''
      x.inputVisible = false
    },
    async commitAlreadyChangeTag (x) {
      const { data: res } = await this.$http.put(`categories/${this.paramId}/attributes/${x.attr_id}`, {
        attr_name: x.attr_name,
        attr_sel: this.tabActiveName,
        attr_vals: x.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('fail in handleInputConfirm')
      } else {
        return this.$message.success('操作请求成功')
      }
    },
    showInput (x) {
      x.inputVisible = true
      this.$nextTick(_ => {
        // console.log('11111111111111zzzzzzzzzzz')
        this.$refs.inputRef.focus()
      })
    },
    closeTag (r, i) {
      r.attr_vals.splice(i, 1)
      this.commitAlreadyChangeTag(r)
    }

  },
  computed: {
    isAbleadd () {
      if (this.selectedvalues.length === 3) {
        return false
      } else {
        return true
      }
    },
    paramId () {
      if (this.selectedvalues.length === 3) {
        return this.selectedvalues[2]
      } else {
        return null
      }
    },
    dialogText () {
      if (this.tabActiveName === 'many') {
        return '添加动态参数'
      } else {
        return '添加静态参数'
      }
    },
    editdialogText () {
      if (this.tabActiveName === 'many') {
        return '动态属性'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-row {
  margin-bottom: 10px;
}

.el-tag {
  margin-right: 5px;
}

.zzwInout {
  width: 150px;
}
</style>
