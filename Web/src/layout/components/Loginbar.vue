<template>
  <div class="Loginbar">
    
    <div class="userbar" v-if="user">
      <el-dropdown>
        <span class="el-dropdown-link">
          <div class="head"><el-image style="width:100%;height:100%;" :src="user.userHead" fit="cover"></el-image></div>
          <div class="username">{{user.userName}}</div>
          <i class="el-icon-arrow-down el-icon--right"></i>
        </span>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="open_photos()">我的相簿</el-dropdown-item>
          <el-dropdown-item @click.native="open_contribute()">我的投稿</el-dropdown-item>
          <el-dropdown-item @click.native="Logout_users">退出登录</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>

    <div class="login" v-else>
        <router-link :to="loginurl" class="link">前往登录 | 注册</router-link>
    </div>

  </div>
</template>
<script>
export default {
  name: "Loginbar",
  data() {
    return {
        user: null,
        loginurl: '/auth/sgin-in'
    };
  },
  created() {
      let user = this.$authUser.getUser()
      console.log(user)
      this.user = user
  },
  methods: {
    open_photos(){
      this.$router.push({ name: "userphotos"})
    },
    open_contribute(){
      this.$router.push({ name: "usercontribute"})
    },
    Logout_users() {
      this.$http
        .authLogout(this.user.userToken)
        .then(response => {
          console.log(response)
          if (response.code === 200) {
            // console.log('>用户退出登录')
            this.$authUser.removeUser()
            window.location.reload();
          }
        })
        .catch(error => {
          console.log("error", error);
        });
    }
  }
};
</script>
<style lang="scss" scoped>
.Loginbar{float: right;}
.login{ 
    height: 60px;line-height: 60px;;
    .link{font-size: 12px;color: #484848}
}
.userbar{
  line-height: 60px;
  .head{float: left;width: 35px;height: 35px;border-radius: 50px;overflow: hidden;margin-top: 15px;}
  .username{float: left;margin-left: 15px;}
  }
</style>