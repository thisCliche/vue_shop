<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/logo.jpg" alt />
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '60px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 侧边栏菜单区 -->
        <el-menu unique-opened background-color="#333744" text-color="#fff" active-text-color="#409eff" :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单的模板区域 -->
            <el-menu-item :index="'/' + item1.path" v-for="item1 in item.children" :key="item1.id" @click="saveNavState('/' + item1.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{item1.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 页面内容主体 -->
      <el-main><router-view></router-view></el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      menuList: [],
      iconObj: {
        '125': 'iconfont icon-user-group',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpingouwudai2',
        '102': 'iconfont icon-danju-tianchong',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    };
  },
  components: {},
  computed: {},
  watch: {},
  methods: {
    logout() {
      this.$confirm('是否确定退出登录？','提示',{
        confirmButtonText:'确定',cancelButtonText: '取消',type: 'warning'
      }).then(() => {
        this.$message.success('退出成功！')
        window.sessionStorage.clear()
        this.$router.push("/login")
      }).catch(()=>{
          this.$message.info('已取消退出！')
      })
    },
    async getMenuList() {
      const {data: res} = await this.$http.get('menus');
      if(res.meta.status != 200) return this.$message.error(res.meta.msg);
      this.menuList = res.data;
    },
    // 菜单折叠
    toggleCollapse() {
      this.isCollapse = !this.isCollapse;
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath',activePath);
      this.activePath = activePath;
    }
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem('activePath');
  },
  mounted() {
  },
};
</script>
<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    img {
      width: 60px;
    }
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background: #333744;
  .toggle-button{background: #4a5064;font-size: 10px;line-height: 24px;color: #fff;text-align: center;letter-spacing: 0.2em;cursor: pointer;}
  .el-menu{border-right: none;}
}
.el-main {
  background: #eaedf1;
}

.iconfont{margin-right: 10px;}
</style>