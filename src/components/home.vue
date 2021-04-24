<template>
  <el-container class="home_contain">
    <el-header>
      <div><img src="../assets/heima.png"> 电商后台管理系统
      </div>
      <el-button type="info" @click="loginOut">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse? '68px':'200px'">
        <div class="buttonCollapse" @click="clickCollapse">|||</div>
        <el-menu :collapse="isCollapse" :collapse-transition="false" :default-active="defaultActive"
                 active-text-color="orange" background-color="#333744" router text-color="#fff" unique-opened>
          <el-submenu v-for="item in leftMenuList" :key="item.id" :index="item.id">
            <template slot="title">
              <i :class='iconFonts[item.id]'></i>
              <span>{{ item.authName }}</span>
            </template>
            <el-menu-item v-for="item2 in item.children" :key="item2.id" :index="'/'+item2.path"
                          @click="saveActive('/'+item2.path)">
              <template><i class="el-icon-menu"></i>
                <span>{{ item2.authName }}</span></template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      leftMenuList: [],
      iconFonts: {
        125: 'iconfont icon-group',
        103: 'iconfont icon-right',
        101: 'iconfont icon-goods',
        102: 'iconfont icon-form',
        145: 'iconfont icon-computer'
      },
      isCollapse: false,
      defaultActive: ''
    }
  },
  created () {
    this.createList()
    this.defaultActive = window.sessionStorage.getItem('path')
  },
  methods: {
    loginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async createList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) {
        return this.$message.error('fail to get the list')
      }
      this.leftMenuList = res.data
      // console.log(this.leftMenuList)
    },
    clickCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveActive (path) {
      this.defaultActive = path
      window.sessionStorage.setItem('path', path)
    }
  }
}
</script>

<style lang="less" scoped>
.home_contain {
  height: 100%;
}

.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 0px;
  font-size: 20px;
  color: #fff;

  > div {
    display: flex;
    align-items: center;

    img {
      margin-right: 10px;
    }
  }
}

.el-aside {
  background-color: #333744;

  .buttonCollapse {
    background-color: #4A5064;
    text-align: center;
    color: #fff;
    letter-spacing: 1px;
    cursor: pointer;
  }

  .el-menu {
    border-right: 0px;
  }

  i {
    margin-right: 6px;
  }
}

.el-main {
  background-color: #eaedf1;
}
</style>
