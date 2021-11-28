<script setup lang="ts">
import { ref } from 'vue'
interface Login {
  account: string,
  password: string,
  remember: boolean
}
let loginForm = ref({
  account: '',
  password: '',
  remember: false
})
</script>
<template>
  <h2>登录</h2>
  <el-form ref="form" :model="loginForm">
    <el-form-item>
      <el-input v-model="loginForm.account"></el-input>
    </el-form-item>
    <el-form-item>
      <el-input v-model="loginForm.password"></el-input>
    </el-form-item>
  </el-form>
  <el-row>
    <el-col :span="12">
      <el-form-item>
        <el-checkbox v-model="loginForm.remember" label="记住我"></el-checkbox>
      </el-form-item>
    </el-col>
    <el-col :span="12" :style="{ 'text-align': 'right' }">
      <el-form-item>
        <a>忘记密码</a>
      </el-form-item>
    </el-col>
  </el-row>
</template>
<style lang="less" scoped>
h2 {
  margin: 0 0 20px 0;
  text-align: center;
}
</style>