<style lang="less">
@import "./login.less";
</style>

<template>
    <div class="login" @keydown.enter="handleSubmit">
        <div class="login-con">
            <Card :bordered="false">
                <p slot="title">
                    <Icon type="log-in"></Icon>
                    欢迎登录
                </p>
                <div class="form-con">
                    <Form ref="loginForm" :model="form" :rules="rules">
                        <FormItem prop="userName">
                            <Input v-model="form.userName" placeholder="请输入用户名">
                                <span slot="prepend">
                                    <Icon :size="16" type="person"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem prop="password">
                            <Input type="password" v-model="form.password" placeholder="请输入密码">
                                <span slot="prepend">
                                    <Icon :size="14" type="locked"></Icon>
                                </span>
                            </Input>
                        </FormItem>
                        <FormItem>
                            <Button @click="handleSubmit" type="primary" long :loading="loading">管理员登录</Button>
                        </FormItem>
                    </Form>
                    <Button @click="kefu" type="success" long>客服登录</Button>
                </div>
            </Card>
        </div>
    </div>
</template>

<script>
import Cookies from "js-cookie";
import login from "./login.js";
export default {
  mixins: [login],
  data() {
    return {
		loading:false,
      form: {
        userName: "",
        password: ""
      },
      rules: {
        userName: [
          { required: true, message: "账号不能为空", trigger: "blur" }
        ],
        password: [{ required: true, message: "密码不能为空", trigger: "blur" }]
      }
    };
  },
  methods: {
    kefu() {
      window.location.href = "https://www.sobot.com/console/home/homeWorkOrder";
    },
    handleSubmit() {
		this.loading=true;
      var params = {
        staffNickname: this.form.userName,
        staffPassword: this.form.password
      };
      this.login(params).then(res => {
        if (res.code == 100000) {
          Cookies.set("user", this.form.userName);
          Cookies.set("password", this.form.password);
          Cookies.set("token", res.data[0].staffToken);
          Cookies.set("staffId", res.data[0].staffId);
          this.$store.commit(
            "setAvator",
            "https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3448484253,3685836170&fm=27&gp=0.jpg"
          );
          this.$router.push({
            name: "home_index"
		  });
		  this.loading=false;
        } else {
			this.loading=false;
          this.$Message.success("登录失败");
        }
      });
      // this.$refs.loginForm.validate((valid) => {
      //     if (valid) {
      //         Cookies.set('user', this.form.userName);
      //         Cookies.set('password', this.form.password);
      //         Cookies.set('token', 'cb9153b08ba74e4d9a4987daee2ff8ba');
      //         Cookies.set('staffId', '21');
      //         this.$store.commit('setAvator', 'https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3448484253,3685836170&fm=27&gp=0.jpg');
      //         if (this.form.userName === 'iview_admin') {
      //             Cookies.set('access', 0);
      //         } else {
      //             Cookies.set('access', 1);
      //         }
      //         this.$router.push({
      //             name: 'home_index'
      //         });
      //     }
      // });
    }
  }
};
</script>

<style>
</style>
