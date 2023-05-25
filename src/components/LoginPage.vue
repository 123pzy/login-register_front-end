<script lang="ts" setup>
import { reactive, ref } from "vue";
import type { FormInstance, FormRules } from "element-plus";
import { useLogin } from "../stores/login";
import axios from "axios";
const login = useLogin();

// 自定义
const already = ref(false);
const style = ref({
  center: true,
});
// 点击登录\注册按钮之后触发:
function changeLogin() {
  already.value = !already.value;
  style.value.center = !style.value.center;
  login.$reset();
}

// 登录:
function submitLogin() {
  if (login.ruleForm.username == "" || login.ruleForm.pass == "") {
    alert("用户名或密码不能为空!");
  } else {
    axios({
      method: "GET",
      url: "/api/login",
      params: {
        username: login.ruleForm.username,
        password: login.ruleForm.pass,
      },
    }).then((res) => {
      alert(res.data);
    });
  }
}

const ruleFormRef = ref<FormInstance>();

const checkAge = (rule: any, value: any, callback: any) => {
  if (!value) {
    return callback(new Error("Please input the age"));
  }
  setTimeout(() => {
    if (!Number.isInteger(value)) {
      callback(new Error("Please input digits"));
    } else {
      if (value < 18) {
        callback(new Error("Age must be greater than 18"));
      } else {
        callback();
      }
    }
  }, 1000);
};

const validatePass = (rule: any, value: any, callback: any) => {
  if (value === "") {
    callback(new Error("请输入密码!"));
  } else {
    if (login.ruleForm.checkPass !== "") {
      if (!ruleFormRef.value) return;
      ruleFormRef.value.validateField("checkPass", () => null);
    }
    callback();
  }
};
const validatePass2 = (rule: any, value: any, callback: any) => {
  if (value === "") {
    callback(new Error("请再次输入密码!"));
  } else if (value !== login.ruleForm.pass) {
    callback(new Error("两次密码输入不一致!"));
  } else {
    callback();
  }
};

const rules = reactive<FormRules>({
  pass: [{ validator: validatePass, trigger: "blur" }],
  checkPass: [{ validator: validatePass2, trigger: "blur" }],
  age: [{ validator: checkAge, trigger: "blur" }],
});

// 验证注册时两次输入的密码是否一致,一致的话发送axios请求,注册用户
const submitForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.validate((valid) => {
    if (valid) {
      console.log("submit!");
      // 注册:
      if (login.ruleForm.username == "") {
        alert("用户名不能为空!");
      } else {
        axios({
          method: "POST",
          url: "/api/register",
          data: {
            username: `${login.ruleForm.username}`,
            password: `${login.ruleForm.pass}`,
          },
        }).then((res) => {
          alert(res.data);
          login.$reset();
        });
      }
    } else {
      console.log("error submit!");
      return false;
    }
  });
};

const resetForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.resetFields();
  // 重置用户名和密码
  login.$reset();
};
</script>

<template>
  <!-- 注册 -->
  <div class="register_container" v-if="already">
    <el-form
      ref="ruleFormRef"
      :model="login.ruleForm"
      status-icon
      :rules="rules"
      label-width="120px"
      class="demo-ruleForm"
    >
      <el-form-item label="用户名" prop="name">
        <el-input v-model="login.ruleForm.username" />
      </el-form-item>
      <el-form-item label="密码" prop="pass">
        <el-input
          v-model="login.ruleForm.pass"
          type="password"
          autocomplete="off"
        />
      </el-form-item>
      <el-form-item label="确认密码" prop="checkPass">
        <el-input
          v-model="login.ruleForm.checkPass"
          type="password"
          autocomplete="off"
        />
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          @click="submitForm(ruleFormRef)"
          :class="style"
          >提交</el-button
        >
        <el-button @click="resetForm(ruleFormRef)">重置</el-button>
      </el-form-item>
      <span class="login_text" @click="changeLogin">已有账号?点我直接登录</span>
    </el-form>
  </div>

  <!-- 登录 -->
  <div class="login_container" v-if="!already">
    <el-form
      ref="ruleFormRef"
      :model="login.ruleForm"
      status-icon
      :rules="rules"
      label-width="120px"
      class="demo-ruleForm"
    >
      <el-form-item label="用户名" prop="name">
        <el-input v-model="login.ruleForm.username" />
      </el-form-item>
      <el-form-item label="密码" prop="pass">
        <el-input
          v-model="login.ruleForm.pass"
          type="password"
          autocomplete="off"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitLogin" :class="style"
          >登录</el-button
        >
      </el-form-item>
      <span class="login_text" @click="changeLogin">还没有账号?点我注册</span>
    </el-form>
  </div>
</template>
<style scoped>
.login_container,
.register_container {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  /* flex-wrap: wrap; */
  align-items: center;
}
.demo-ruleForm {
  width: 25vw;
  min-width: 400px;
}
.login_text {
  display: block;
  width: 100%;
  padding-left: 50px;
  text-align: center;
}
.center {
  width: 500px;
  margin-left: auto;
  margin-right: auto;
}
</style>
