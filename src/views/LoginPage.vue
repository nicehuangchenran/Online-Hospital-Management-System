<template>
  <div class="logincontent">
    <div class="centered-text">济康同行</div>
    <div class="centered-text-2">Online Hospital Management System</div>
    <div class="login-box">
      <el-form
        ref="loginFormRef"
        :model="loginForm"
        :rules="loginFormRules"
        class="login-form"
        label-position="left"
        label-width="80px"
      >
        <h3 class="login-title">登录</h3>

        <el-form-item :prop="username" class="input-box" label="账号">
          <el-tooltip
            class="item"
            effect="dark"
            content="账号即为注册时的ID号"
            placement="right"
          >
            <el-input
              type="text"
              v-model="loginForm.username"
              auto-complete="off"
              placeholder="请输入账号"
            ></el-input>
          </el-tooltip>
        </el-form-item>

        <el-form-item :prop="password" class="input-box" label="密码">
          <el-input
            type="password"
            v-model="loginForm.password"
            auto-complete="off"
            placeholder="请输入密码"
          ></el-input>
        </el-form-item>

        <div class="rec-group">
          <div
            class="rec"
            v-for="(role_text, role_index) in roles_text"
            :label="role_text"
            :key="role_index"
            @click="roleChange(role_index)"
            :class="{ active: role_index === role_num }"
          >
            {{ role_text }}
          </div>
        </div>

        <div class="button-group">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="primary" @click="toRegister">注册</el-button>
        </div>

        <div class="bottom-actions">
          <el-radio
            v-model="auto_login"
            :label="true"
            @click.prevent="onRadioChange"
            >3天内自动登录</el-radio
          >
          <el-button type="text" @click="toForget">忘记密码</el-button>
        </div>
      </el-form>
    </div>
    <div class="el-login-footer">
      <span>Copyright © 2018-2023 济康同行 All Rights Reserved.</span>
    </div>
  </div>
</template>

  <script>
import axios from "axios";
import { useRouter } from "vue-router";
import { useStore } from "vuex";

export default {
  name: "LoginPage",
  data() {
    return {
      router: useRouter(),
      store: useStore(),
      access_token: null,
      auto_login: false,
      headers: {},

      //登录表单数据绑定
      loginForm: {
        username: "",
        password: "",
      },
      //表单的验证规则
      loginFormRules: {
        //验证用户ID是否合法
        username: [
          { required: true, message: "请输入账号", trigger: "blur" },
          {
            min: 11,
            max: 11,
            message: "用户账号必须是11位",
            trigger: "blur",
          },
        ],
        //    验证密码是否合法
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          {
            min: 1,
            max: 50,
            message: "密码必须在1-50个字符之间",
            trigger: "blur",
          },
        ],
        userToken: "",
      },
      //是否保持登录
      isStayLogin: false,
      //身份多选框文字
      roles_text: ["普通用户", "医生", "管理员"],
      role_checkBox: "普通用户",
      role_num: "0",
      administratorId: "23202",
    };
  },
  //回车登录操作
  created() {
    let UserList = ["Patient", "Doctor", "Administrator"];
    let AfterLogin = ["/Patient", "/doctor-operator", "/Admin"];

    axios
      .get("http://124.223.143.21/api/Login/verifyToken", {
        params: {
          User: UserList[this.role_num],
          token: this.getCookie("token"),
        },
      })
      .then((response) => {
        this.$message.success("正在自动登录");
        if (response.data) {
          console.log("登录成功");
          setTimeout(() => {
            this.router.push({
              path: AfterLogin[this.getCookie("role")],
            });
          }, 2000);

          sessionStorage.setItem("userID", this.getCookie("ID"));
          sessionStorage.setItem("role", this.getCookie("role"));
          return;
        } else {
          // 登录失败
          console.error("登录失败");
          this.$msgbox.alert("用户名或密码错误，登录失败");
        }
      })
      .catch((error) => {
        console.error(error);
      });
  },
  methods: {
    setCookie(name, value, exdays) {
      var d = new Date();
      d.setTime(d.getTime() + exdays * 24 * 60 * 60 * 1000);
      var expires = "expires=" + d.toGMTString();
      document.cookie = name + "=" + value + "; " + expires;
    },

    getCookie(name) {
      let ret, m;
      if (typeof name === "string" && name !== "") {
        if (
          (m = String(document.cookie).match(
            new RegExp("(?:^| )" + name + "(?:(?:=([^;]*))|;|$)")
          ))
        ) {
          ret = m[1] ? decodeURIComponent(m[1]) : "";
        }
      }
      return ret;
    },

    login() {
      let LoginURL = ["PatientLogin", "DoctorLogin", "AdminLogin"];
      let AfterLogin = ["/Patient", "/doctor-operator", "/Admin"];
      axios
        .get(
          "http://124.223.143.21/api/Login/" + LoginURL[this.role_num] + "/",
          {
            params: {
              ID: this.loginForm.username,
              password: this.loginForm.password,
            },
          }
        )
        .then((response) => {
          console.log(response.data.slice(14));
          if (response.data) {
            console.log("登录成功");
            this.router.push({
              path: AfterLogin[this.role_num],
            });

            sessionStorage.setItem("userID", this.loginForm.username);
            sessionStorage.setItem("role", this.role_num);

            if (this.auto_login) {
              this.setCookie("token", response.data.slice(14), 3);
              this.setCookie("ID", this.loginForm.username, 3);
              this.setCookie("role", this.role_num, 3);
            }
          } else {
            // 登录失败
            console.error("登录失败");
            this.$msgbox.alert("用户名或密码错误，登录失败")
            .catch(e => {
                console.error("Error with message box:", e);
            });

          }
        })
        .catch((error) => {
          console.error(error);
          this.$msgbox.alert("用户名或密码错误，登录失败")
            .catch(e => {
              console.error("Error with message box:", e);
          });
        });
    },

    toRegister() {
      this.router.push({
        path: "/register",
      });
    },

    roleChange(index) {
      this.role_num = index;
      this.role_checkBox = this.roles_text[index];
      console.log(this.role_num);
      console.log(this.role_checkBox);
    },

    onRadioChange() {
      console.log(this.auto_login);
      this.auto_login = !this.auto_login;
    },

    toForget() {
      this.router.push({
        path: "/forget",
      });
    },
  },
};
</script>

<style lang="less" scoped>
.logincontent {
  display: flex;
  flex-direction: column;
  background-image: url("../assets/login-background.jpg");
  background-size: 100% 100%;
  background-attachment: fixed;

  width: 100%;
  height: 100%;
  min-width: 900px;
  min-height: 1000px;

  justify-content: center;
  align-items: center;
}
.centered-text {
  position: absolute;
  top: 20%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 4em;
  font-weight: bold;
  z-index: 2;
}
.centered-text-2 {
  position: absolute;
  top: 28%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 2em;
  font-weight: bold;
  z-index: 2;
  // font-family: "楷体", KaiTi, sans-serif;
}
.login-box {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  width: 400px;
  padding: 20px;
  margin: 50px auto;
  background-color: #f3f3f3;
  border: 1px solid #d0d0d0;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);

  .login-form {
    display: flex;
    flex-direction: column;

    .login-title {
      text-align: center;
      // font-family: "Microsoft YaHei";
      font-weight: bold;
      font-size: 25px;
      color: #002fa7; /* Klein Blue */
      margin-bottom: 20px;
    }

    .input-box .el-input {
      border-radius: 5px;
      border-color: #002fa7; /* Klein Blue */
      &:hover {
        border-color: darken(#002fa7, 10%);
      }
    }

    .rec-group {
      display: flex;
      justify-content: center;
      margin-bottom: 20px;
      .rec {
        display: flex;
        background-color: white;
        width: 70px;
        height: 30px;
        border-radius: 10%;
        color: #2c3e50; /* Klein Blue */
        align-items: center;
        justify-content: center;
        font-size: 1px;
        margin: 0 10px;
        border: 1px solid #dae4f4;
        &:hover {
          background-color: #89bcef;
          color: white;
          opacity: 0.4;
        }
        &.active {
          background-color: #002fa7;
          color: white;
        }
      }
    }

    .button-group {
      display: flex;
      justify-content: space-between;
      margin-top: 20px;

      .el-button {
        background-color: #002fa7; /* Klein Blue */
        border-color: #002fa7; /* Klein Blue */
        &:hover {
          background-color: darken(#002fa7, 10%);
          border-color: darken(#002fa7, 10%);
        }
      }
    }

    .bottom-actions {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: 20px;
    }
  }
}
.el-login-footer {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #fff;
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
}
* {
  font-family: AliRegular;
  --va-font-family: AliRegular;
  /* 应用字体 */
}
</style>