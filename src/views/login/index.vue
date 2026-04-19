<template>
  <div class="login-container">
    <div class="login-content">
      <div class="login-info">
        <p class="title">欢迎登录 在线教育系统</p>
        <p class="info">
          在线课程匿名化评价系统系统旨在解决当前中心化评价平台存在的“刷好评/差评”、“评价被删改”、“用户因怕得罪创作者而不敢直言”等痛点
        </p>
      </div>
      <div class="login-panel">
        <el-form v-loading="loading" :model="loginForm" label-position="left" @keyup.enter="handleLogin()">
          <h3 class="login-head">安全管理登录</h3>
          <el-form-item class="form-group" prop="mobile">
            <el-input v-model="loginForm.mobile" placeholder="管理员账号" />
          </el-form-item>
          <el-form-item class="form-group" prop="mobilePwd">
            <el-input v-model="loginForm.mobilePwd" placeholder="登录密码" type="password" show-password />
          </el-form-item>
          <el-form-item class="form-group" prop="verCode">
            <el-input v-model="loginForm.verCode" class="var-input" placeholder="验证码" />
            <img class="var-img" :src="verImg" @click="getCaptcha" />
          </el-form-item>
          <el-button class="login-button" type="primary" @click="handleLogin">安全登录</el-button>
        </el-form>
      </div>
    </div>
    <div class="footer">
      <div v-if="service.websiteCopyright" class="copyright">
        <span v-html="service.websiteCopyright" />
      </div>
      <div>
        <a v-if="service.websiteIcp" href="http://beian.miit.gov.cn/" target="_blank">{{ service.websiteIcp }}</a>
        <span>&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;</span>
        <img v-if="service.websitePrn" class="website-prn" :alt="service.websitePrn" src="../../assets/images/common/beian.png" />
        <a v-if="service.websitePrn" :href="'http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=' + service.websitePrnNo" target="_blank">&nbsp;{{ service.websitePrn }} </a>
      </div>
    </div>
  </div>
</template>

<script setup>
  import { loginApi } from '@/api/login'
  import { onMounted, reactive, ref } from 'vue'
  import { setToken } from '@/utils/cookie'
  import { useRouter } from 'vue-router'
  import { useUserStore } from '@/store/modules/user'
  import { createNewRouter } from '@/router'
  import { PATH } from '@/utils/constants/system'
  import { ElMessage } from 'element-plus'
  import { encrypt } from '@/utils/base.js'

  const router = useRouter()
  const loading = ref(false)
  const verImg = ref()

  // 站点信息
  const service = ref({})
  onMounted(() => {
    loginApi.getWebsite().then((res) => {
      service.value = res
    })
    // 验证码
    getCaptcha()
  })

  // 登录
  const loginForm = reactive({
    mobile: '13300000000',
    mobilePwd: '123456'
  })

  // 获取验证码
  async function getCaptcha() {
    try {
      const res = await loginApi.getCodeImg()
      loginForm.verToken = res.verToken
      verImg.value = res.img
    } catch (error) {
      console.error(error)
    }
  }

  async function handleLogin() {
    if (!loginForm.verCode) {
      ElMessage.error('请输入验证码')
      return
    }

    loading.value = true
    try {
      // 密码加密
      loginForm.mobilePwdEncrypt = encrypt(loginForm.mobilePwd, service.value.rsaLoginPublicKey)
      delete loginForm.mobilePwd
      const res = await loginApi.login(loginForm)
      // 存入cookie
      setToken(res.token)
      // 更新store
      useUserStore().login(res)
      // 初始化路由
      createNewRouter(res.routerList)
      await router.push(PATH.URL_DASHBOARD)
    } catch (error) {
      console.error(error)
      await getCaptcha()
    } finally {
      loading.value = false
    }
  }
</script>

<style lang="scss" scoped>
  .login-container {
    height: 100vh;
    background: #2873f0;
  }

  .login-content {
    position: absolute;
    top: calc((100vh - 520px) / 2);
    left: 0;
    right: 0;
    width: 800px;
    margin: 0 auto;

    .login-info {
      float: left;
      width: 350px;
      padding: 25px;
      height: 450px;
      color: #fff;
      background: #033c94;
      border-radius: 12px 0 0 12px;

      .title {
        font-size: 20px;
        font-weight: 700;
      }

      .info {
        font-size: 15px;
        font-weight: 300;
        line-height: 25px;
      }
    }

    .login-panel {
      float: right;
      background: #fff;
      height: 450px;
      width: 350px;
      padding: 25px;
      border-radius: 0 12px 12px 0;

      .login-head {
        text-align: center;
        font-size: 28px;
        font-weight: 700;
      }

      .login-button {
        width: 350px;
        height: 45px;
        background: #2873f0;
        border-radius: 4px;
        font-size: 16px;
        font-weight: 700;
        text-align: center;
        color: #fff;
        line-height: 50px;
        cursor: pointer;
      }

      .tip {
        margin-top: 20px;
        font-size: 14px;
      }
    }
  }

  .footer {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    color: #fff;
    text-align: center;
    padding-bottom: 20px;
    font-size: 13px;
    line-height: 25px;

    a {
      text-decoration: none;
      color: #fff;
    }
    .website-prn {
      width: auto;
    }
  }

  .var-input {
    width: 200px;
  }

  .var-img {
    margin-left: 20px;
    width: 80px;
    height: auto;
  }

  .el-input {
    height: 40px;
  }
</style>
