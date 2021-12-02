<script setup lang="ts">
import { ref } from 'vue'
import LoginForm from './LoginForm.vue'
let register: boolean = false
</script>
<template>
  <div class="login">
    <div
      v-show="!register"
      class="bc-login-form"
    >
      <LoginForm />
    </div>
    <div
      v-show="register"
      class="bc-login-form"
    >
      <h2>注册</h2>
    </div>
  </div>
</template>
<style lang="less" scoped>
@import '@/styles/public.less';
.login {
  min-height: 100%;
  background-color: @login-bg-normal;
  background-image: @login-bg-image;
  background-repeat: no-repeat;
  background-size: cover;
  display: flex;
  align-items: center;
  justify-content: center;
}
.bc-login-form {
  background-color: #fff;
  width: 400px;
  height: auto;
  border-radius: 5px;
  min-height: 200px;
  padding: 20px 30px;
}
</style>