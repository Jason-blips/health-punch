<template>
  <el-card>
    <el-container>
      <el-card class="box-card">
        <el-header>
          <el-menu :default-active="this.$route.path" routerclass="el-menu-demo" mode="horizontal"
                   @select="handleSelect" router>
            <a href='https://cxq21.gitee.io' target="_blank"><img src="../assets/LIEFox.png"
                                                                  style="width: 40px; height: 50px;float:left;margin-top: 5px;margin-right:50px"></a>
            <el-menu-item v-for="(item,i) in navList" :key="i" :index="item.name">
              {{ item.navItem }}
            </el-menu-item>
            <el-dropdown style="float:right;margin-top: 5px">
              <el-avatar style="background: #17af74;" class="el-dropdown-link" :size="50"> {{ showUsername.substr(0,3) }}</el-avatar>
              <el-dropdown-menu slot="dropdown">
                <router-link to="/personInfo">
                  <el-dropdown-item icon="el-icon-user">个人信息</el-dropdown-item>
                </router-link>
                <router-link to="/">
                  <el-dropdown-item icon="el-icon-lock">退出</el-dropdown-item>
                </router-link>
              </el-dropdown-menu>
            </el-dropdown>
          </el-menu>
        </el-header>
      </el-card>
      <el-main>
        <router-view></router-view>
      </el-main>
      <el-footer>
        <div style="justify-content: flex-end;align-items: stretch">
          <a href='https://gitee.com/cxq21/health-punch-vs' target="_blank">
            <img src='https://gitee.com/cxq21/health-punch-vs/widgets/widget_6.svg' alt='Fork me on Gitee'></img>
          </a>
        </div>
      </el-footer>
    </el-container>
  </el-card>
</template>

<script>
import Cookies from "js-cookie"

export default {
  name: 'App',
  data() {
    return {
      navList: [
        {name: '/main', navItem: '健康打卡'},
        {name: '/logs', navItem: '打卡记录'},
        // {name: '/data', navItem: '数据大屏'},
      ],
      // xians: this.$route.path.includes('login'),
    };
  },
  computed: {
    showUsername() {
      return Cookies.get("username")
    }
  },
  methods: {
    handleSelect(key, keyPath) {
      console.log(key, keyPath);
    },
  },
}
</script>

<style scoped>
.el-header, .el-footer {
  text-align: center;
  line-height: 60px;
}

a {
  text-decoration: none;
}

.router-link-active {
  text-decoration: none;
}
</style>