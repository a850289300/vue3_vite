<script setup lang="ts">
import { ref } from 'vue'
interface Login {
  account: string
  password: string
  remember: boolean
}
let loginForm = ref({
  account: '',
  password: '',
  remember: false,
})

const rules = {
  account: [
    {
      required: true,
      message: '账号不能为空',
      trigger: 'blur',
    },
  ],
  password: [
    {
      required: true,
      message: '密码不能为空',
      trigger: 'blur',
    },
  ],
}
</script>
<template>
  <h2>登录</h2>
  <el-form
    ref="form"
    :model="loginForm"
    :rules="rules"
  >
    <el-form-item prop="account">
      <el-input v-model="loginForm.account"></el-input>
    </el-form-item>
    <el-form-item prop="password">
      <el-input v-model="loginForm.password"></el-input>
    </el-form-item>
    <el-row>
      <el-col :span="12">
        <el-form-item>
          <el-checkbox
            v-model="loginForm.remember"
            label="记住我"
          ></el-checkbox>
        </el-form-item>
      </el-col>
      <el-col
        :span="12"
        class="text-right"
      >
        <el-form-item>
          <a :style="{ 'cursor': 'pointer', 'color': '#0960bd' }">忘记密码?</a>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="18">
        <el-form-item>
          <el-button
            type="primary"
            size="medium"
          >登录</el-button>
        </el-form-item>
      </el-col>
      <el-col
        :span="6"
        class="text-right"
      >
        <el-form-item>
          <el-button
            type="default"
            size="medium"
          >注册</el-button>
        </el-form-item>
      </el-col>
    </el-row>
  </el-form>
</template>
<style lang="less" scoped>
h2 {
  margin: 0 0 20px 0;
  text-align: center;
}
.el-button {
  width: 100%;
}
</style>