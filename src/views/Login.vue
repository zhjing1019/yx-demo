<template>
    <div>
        <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="名称" prop="account">
            <el-input v-model="ruleForm.account"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
            <el-input v-model="ruleForm.password"></el-input>
        </el-form-item>
        <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">登录</el-button>
        </el-form-item>
        </el-form>
    </div>
</template>

<script>
import md5 from './../utils/md5'
import cookie from './../utils/cookie'
// import config from './../configs'
  export default {
    data() {
      return {
        ruleForm: {
          account: '',
          password: '',
        },
        rules: {
          account: [
            { required: true, message: '请输入名称', trigger: 'blur' },
          ],
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
          ],
          
        }
      };
    },
    methods: {
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            let sdktoken = md5(this.ruleForm.password)
            // 服务端帐号均为小写
            cookie.setCookie('uid', this.ruleForm.account.toLowerCase());
            cookie.setCookie('sdktoken', sdktoken);
            this.$router.push('Home');
          } else {
            return false;
          }
        });
      },

    }
  }
</script>