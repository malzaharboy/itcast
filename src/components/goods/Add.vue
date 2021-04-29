<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!--  卡片区-->
    <el-card>
      <el-alert center show-icon title="添加商品信息" type="info"></el-alert>
      <!--    steps-->
      <el-steps :active="parseInt(currentValue)" :space="200" align-center finish-status="success">
        <el-step title="基础信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!--   tabs -->
      <el-form ref="commitFormRef" :model="commitForm" :rules="Formrules" label-position="top" label-width="100px">
        <el-tabs v-model="currentValue" :before-leave="changeTab" tab-position="left" @tab-click="tabChangeClick">
          <el-tab-pane label="基础信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="commitForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="commitForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="commitForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="commitForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类">
              <el-cascader v-model="selectedlist" :options="paramsList" :props="cascadeProps" expand-trigger="hover"
                           @change="cascadeChange">
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyData" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox v-for="(item2 ,i) in item.attr_vals" :key="i" :label="item2" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item in onlyData" :key="item.attr_id" :label="item.attr_name">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload :action="upLoadUrl" :file-list="uploadFileList" :headers="headers"
                       :on-preview="uploadPreview" :on-remove="uploadRemove"
                       :on-success="uploadSuccess" list-type="picture">
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor ref="myQuillEditor" v-model="commitForm.goods_introduce"/>
            <el-button type="primary" @click="commitAllAdd">添加商品</el-button>
          </el-tab-pane>
          <el-tab-pane label="完成" name="5">定时任务补偿</el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <el-dialog
      :visible.sync="dialogVisible" title="提示" width="30%">
      <img :src="prewviewImg" style="width: 100%;">
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      currentValue: '0',
      commitForm: {
        goods_name: '',
        goods_price: 0,
        goods_number: 0,
        goods_weight: 0,
        pics: [],
        goods_cat: '',
        attrs: [],
        goods_introduce: ''
      },
      Formrules: {
        goods_name: [{
          required: 'true',
          message: '请输入名称',
          trigger: 'blur'
        }],
        goods_price: [{
          min: 0,
          message: 'cant be 0 or less than 0',
          trigger: 'blur',
          transform (value) {
            if (value === 0) {
              return -1
            } else {
              return value
            }
          }
        }],
        goods_weight: [{
          min: 0,
          message: 'cant be 0 or less than 0',
          trigger: 'blur',
          transform (value) {
            if (value === 0) {
              return -1
            } else {
              return value
            }
          }
        }],
        goods_number: [{
          min: 0,
          message: 'cant be 0 or less than 0',
          trigger: 'blur',
          transform (value) {
            if (value === 0) {
              return -1
            } else {
              return value
            }
          }
        }]
      },
      paramsList: [],
      cascadeProps: {
        children: 'children',
        value: 'cat_id',
        label: 'cat_name'
      },
      selectedlist: [],
      manyData: [],
      onlyData: [],
      upLoadUrl: 'http://127.0.0.1:8888/api/private/v1/upload/',
      uploadFileList: [],
      headers: { Authorization: window.sessionStorage.getItem('token') },
      dialogVisible: false,
      prewviewImg: ''
    }
  },
  methods: {
    async getParamsList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('fail in getParamsList')
      }
      this.paramsList = res.data
    },
    cascadeChange () {
      if (this.selectedlist.length < 3) {
        this.selectedlist = []
      }
      console.log()
    },
    changeTab (newt, oldt) {
      if (oldt === '0' && this.selectedlist.length < 3) {
        this.$message.error('请填写选择商品分类')
        return false
      }
      return true
    },
    async tabChangeClick () {
      if (this.currentValue === '1') {
        const { data: res } = await this.$http.get(`categories/${this.currentCatId}/attributes`, {
          params: {
            sel: 'many'
          }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('fail in tabChangeClick')
        }
        this.manyData = res.data
        this.manyData.forEach(item => {
          if (item.attr_vals) {
            item.attr_vals = item.attr_vals.split(' ')
          } else {
            item.attr_vals = []
          }
          item.arr = []
        })
        console.log(this.manyData)
      }
      if (this.currentValue === '2') {
        const { data: res } = await this.$http.get(`categories/${this.currentCatId}/attributes`, {
          params: {
            sel: 'only'
          }
        })
        if (res.meta.status !== 200) {
          return this.$message.error('fail in tabChangeClick')
        }
        this.onlyData = res.data
        console.log(this.onlyData)
      }
    },
    uploadSuccess (response, file, filelist) {
      // console.log(response)
      this.commitForm.pics.push({ pic: response.data.tmp_path })
      // console.log(this.commitForm)
    },
    uploadRemove (file, filelist) {
      // console.log(file, filelist, this.commitForm.pics)
      const index = this.commitForm.pics.findIndex(item => {
        return item.pic === file.response.data.tmp_path
      })
      this.commitForm.pics.splice(index, 1)
    },
    uploadPreview (file) {
      this.prewviewImg = file.url
      this.dialogVisible = true
      // console.log(file)
    },
    commitAllAdd () {
      this.$refs.commitFormRef.validate(async val => {
        if (!val) {
          return this.$message.error('please check the content')
        }
        this.commitForm.goods_cat = this.selectedlist.join(',')
        for (const item of this.manyData) {
          const x = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.commitForm.attrs.push(x)
        }
        for (const item of this.onlyData) {
          const x = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.commitForm.attrs.push(x)
        }
        const { data: res } = await this.$http.post('goods', this.commitForm)
        if (res.meta.status !== 201) {
          console.log(res)
          return this.$message.error('error in the  commitAllAdd')
        }
        this.$message.success('添加商品成功')
        console.log(this.manyData, this.onlyData)
      })
    }

  },
  created () {
    this.getParamsList()
  },
  computed: {
    currentCatId () {
      if (this.selectedlist < 3) {
        return ''
      } else {
        return this.selectedlist[this.selectedlist.length - 1]
      }
    }
  }
}
</script>

<style scoped>
.el-steps {
  margin: 15px 0;
}

.el-form-item .el-input {
  width: 70%;
}

.el-checkbox {
  margin: 0 8px 0 0 !important;
}
</style>
