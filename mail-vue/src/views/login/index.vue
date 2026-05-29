<template>
  <div id="login-box" :style="background ? backgroundStyle : ''" v-loading="oauthLoading" element-loading-text="登录中...">

    <!-- Claude-style ambient background (only when no custom bg) -->
    <div v-if="!settingStore.settings.background" class="ambient-bg">
      <div class="ambient-orb orb-1"></div>
      <div class="ambient-orb orb-2"></div>
      <div class="ambient-orb orb-3"></div>
      <div class="grain-overlay"></div>
    </div>

    <!-- Left panel: branding -->
    <div class="brand-panel" v-if="!settingStore.settings.background">
      <div class="brand-content">
        <div class="brand-logo">
          <div class="logo-icon">
            <Icon icon="mdi:email-outline" width="28" height="28" />
          </div>
          <span class="logo-text">GOPK Mail</span>
        </div>
        <p class="brand-tagline">
          A thoughtful space<br>for your correspondence.
        </p>
        <div class="brand-features">
          <div class="feature-item">
            <Icon icon="solar:shield-check-linear" width="16" height="16" />
            <span>Serverless & secure</span>
          </div>
          <div class="feature-item">
            <Icon icon="solar:bolt-linear" width="16" height="16" />
            <span>Cloudflare-powered</span>
          </div>
          <div class="feature-item">
            <Icon icon="solar:devices-linear" width="16" height="16" />
            <span>Works on every device</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Right panel: form -->
    <div class="form-wrapper">
      <div class="container">

        <!-- Logo (shown when custom bg is active) -->
        <div v-if="settingStore.settings.background" class="form-logo">
          <div class="logo-icon-sm">
            <Icon icon="mdi:email-outline" width="18" height="18" />
          </div>
          <span>GOPK Mail</span>
        </div>

        <!-- Header -->
        <div class="form-header">
          <h1 class="form-title">
            <template v-if="show === 'login'">Welcome back</template>
            <template v-else>Create an account</template>
          </h1>
          <p class="form-desc">
            <template v-if="show === 'login'">{{ $t('loginTitle') }}</template>
            <template v-else>{{ $t('regTitle') }}</template>
          </p>
        </div>

        <!-- ── Login form ── -->
        <div v-show="show === 'login'" class="form-fields">
          <div class="field-group">
            <label class="field-label">{{ $t('emailAccount') }}</label>
            <el-input
              :class="!hideLoginDomain ? 'email-input' : ''"
              v-model="form.email"
              type="text"
              :placeholder="$t('emailAccount')"
              autocomplete="off"
            >
              <template #append v-if="!hideLoginDomain">
                <div @click.stop="openSelect">
                  <el-select
                    v-if="show === 'login'"
                    ref="mySelect"
                    v-model="suffix"
                    :placeholder="$t('select')"
                    class="select"
                  >
                    <el-option v-for="item in domainList" :key="item" :label="item" :value="item" />
                  </el-select>
                  <div class="suffix-display">
                    <span>{{ suffix }}</span>
                    <Icon icon="mingcute:down-small-fill" width="16" height="16"/>
                  </div>
                </div>
              </template>
            </el-input>
          </div>

          <div class="field-group">
            <label class="field-label">{{ $t('password') }}</label>
            <el-input v-model="form.password" :placeholder="$t('password')" type="password" autocomplete="off" />
          </div>

          <el-button class="btn btn-primary" @click="submit" :loading="loginLoading">
            {{ $t('loginBtn') }}
          </el-button>

          <div v-if="settingStore.settings.linuxdoSwitch" class="divider">
            <span>or</span>
          </div>

          <el-button v-if="settingStore.settings.linuxdoSwitch" class="btn btn-oauth" @click="linuxDoLogin">
            <el-avatar src="/image/linuxdo.webp" :size="16" style="margin-right: 8px" />
            Continue with LinuxDo
          </el-button>
        </div>

        <!-- ── Register form ── -->
        <div v-show="show !== 'login'" class="form-fields">
          <div class="field-group">
            <label class="field-label">{{ $t('emailAccount') }}</label>
            <el-input
              :class="!hideLoginDomain ? 'email-input' : ''"
              v-model="registerForm.email"
              type="text"
              :placeholder="$t('emailAccount')"
              autocomplete="off"
            >
              <template #append v-if="!hideLoginDomain">
                <div @click.stop="openSelect">
                  <el-select
                    v-if="show !== 'login'"
                    ref="mySelect"
                    v-model="suffix"
                    :placeholder="$t('select')"
                    class="select"
                  >
                    <el-option v-for="item in domainList" :key="item" :label="item" :value="item" />
                  </el-select>
                  <div class="suffix-display">
                    <span>{{ suffix }}</span>
                    <Icon icon="mingcute:down-small-fill" width="16" height="16"/>
                  </div>
                </div>
              </template>
            </el-input>
          </div>

          <div class="field-group">
            <label class="field-label">{{ $t('password') }}</label>
            <el-input v-model="registerForm.password" :placeholder="$t('password')" type="password" autocomplete="off"/>
          </div>

          <div class="field-group">
            <label class="field-label">{{ $t('confirmPwd') }}</label>
            <el-input v-model="registerForm.confirmPassword" :placeholder="$t('confirmPwd')" type="password" autocomplete="off"/>
          </div>

          <div class="field-group" v-if="settingStore.settings.regKey === 0">
            <label class="field-label">{{ $t('regKey') }}</label>
            <el-input v-model="registerForm.code" :placeholder="$t('regKey')" type="text" autocomplete="off"/>
          </div>

          <div class="field-group" v-if="settingStore.settings.regKey === 2">
            <label class="field-label">{{ $t('regKeyOptional') }}</label>
            <el-input v-model="registerForm.code" :placeholder="$t('regKeyOptional')" type="text" autocomplete="off"/>
          </div>

          <div v-show="verifyShow"
               class="register-turnstile"
               :data-sitekey="settingStore.settings.siteKey"
               data-callback="onTurnstileSuccess"
               data-error-callback="onTurnstileError"
               data-after-interactive-callback="loadAfter"
               data-before-interactive-callback="loadBefore"
          >
            <span style="font-size: 12px;color: #E05252" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
          </div>

          <el-button class="btn btn-primary" @click="submitRegister" :loading="registerLoading">
            {{ $t('regBtn') }}
          </el-button>

          <div v-if="settingStore.settings.linuxdoSwitch" class="divider"><span>or</span></div>
          <el-button v-if="settingStore.settings.linuxdoSwitch" class="btn btn-oauth" @click="linuxDoLogin">
            <el-avatar src="/image/linuxdo.webp" :size="16" style="margin-right: 8px" />
            Continue with LinuxDo
          </el-button>
        </div>

        <!-- Switch login / register -->
        <template v-if="settingStore.settings.register === 0">
          <div class="switch" @click="show = 'register'" v-if="show === 'login'">
            {{ $t('noAccount') }} <span>{{ $t('regSwitch') }}</span>
          </div>
          <div class="switch" @click="show = 'login'" v-else>
            {{ $t('hasAccount') }} <span>{{ $t('loginSwitch') }}</span>
          </div>
        </template>
      </div>
    </div>

    <!-- Bind dialog (OAuth) -->
    <el-dialog class="bind-dialog gopk-dialog" v-model="showBindForm" title="绑定邮箱">
      <div class="bind-container">
        <el-input :class="!hideLoginDomain ? 'email-input' : ''" v-model="bindForm.email" type="text" :placeholder="$t('emailAccount')" autocomplete="off">
          <template #append v-if="!hideLoginDomain">
            <div @click.stop="openSelect">
              <el-select ref="mySelect" v-model="suffix" :placeholder="$t('select')" class="select">
                <el-option v-for="item in domainList" :key="item" :label="item" :value="item" />
              </el-select>
              <div class="suffix-display">
                <span>{{ suffix }}</span>
                <Icon icon="mingcute:down-small-fill" width="16" height="16"/>
              </div>
            </div>
          </template>
        </el-input>
        <el-input v-if="settingStore.settings.regKey === 0" v-model="bindForm.code" :placeholder="$t('regKey')" type="text" autocomplete="off"/>
        <el-input v-if="settingStore.settings.regKey === 2" v-model="bindForm.code" :placeholder="$t('regKeyOptional')" type="text" autocomplete="off"/>
        <el-button class="btn btn-primary" @click="bind" :loading="bindLoading">绑定</el-button>
      </div>
    </el-dialog>

    <!-- GitHub link -->
    <a v-show="settingStore.settings.projectLink" class="github-link" href="https://github.com/maillab/cloud-mail" target="_blank">
      <Icon icon="mingcute:github-line" width="16" height="16" />
    </a>
  </div>
</template>

<script setup>
import router from "@/router";
import {computed, nextTick, reactive, ref} from "vue";
import {login} from "@/request/login.js";
import {register} from "@/request/login.js";
import {websiteConfig} from "@/request/setting.js";
import {isEmail} from "@/utils/verify-utils.js";
import {useSettingStore} from "@/store/setting.js";
import {useAccountStore} from "@/store/account.js";
import {useUserStore} from "@/store/user.js";
import {useUiStore} from "@/store/ui.js";
import {Icon} from "@iconify/vue";
import {cvtR2Url} from "@/utils/convert.js";
import {loginUserInfo} from "@/request/my.js";
import {permsToRouter} from "@/perm/perm.js";
import {useI18n} from "vue-i18n";
import {oauthBindUser, oauthLinuxDoLogin} from "@/request/ouath.js";

const {t} = useI18n();
const accountStore = useAccountStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const settingStore = useSettingStore();
const loginLoading = ref(false)
const bindLoading = ref(false)
const oauthLoading = ref(false);
const showBindForm = ref(false);
const show = ref('login')

const bindForm = reactive({ email: '', oauthUserId: '', code: '' })
const form = reactive({ email: '', password: '' });
const mySelect = ref()
const suffix = ref('')
const registerForm = reactive({ email: '', password: '', confirmPassword: '', code: null })
const domainList = settingStore.domainList;
const registerLoading = ref(false)
suffix.value = domainList[0]
const verifyShow = ref(false)
let verifyToken = ''
let turnstileId = null
let botJsError = ref(false)
let verifyErrorCount = 0

window.onTurnstileSuccess = (token) => { verifyToken = token; };
window.onTurnstileError = (e) => {
  if (verifyErrorCount >= 4) return;
  verifyErrorCount++;
  setTimeout(() => {
    nextTick(() => {
      if (!turnstileId) { turnstileId = window.turnstile.render('.register-turnstile') }
      else { window.turnstile.reset(turnstileId); }
    })
  }, 1500)
};
window.loadAfter = () => {};
window.loadBefore = () => {};

const hideLoginDomain = computed(() => settingStore.settings.loginDomain === 1)

const backgroundStyle = computed(() => {
  return settingStore.settings.background ? {
    'background-image': `url(${cvtR2Url(settingStore.settings.background)})`,
    'background-repeat': 'no-repeat',
    'background-size': 'cover',
    'background-position': 'center'
  } : {}
})

const background = computed(() => settingStore.settings.background)

const openSelect = () => { mySelect.value.toggleMenu() }
const getFullEmail = (email) => hideLoginDomain.value ? email : email + suffix.value
const getEmailName = (email) => email.split('@')[0]

function linuxDoLogin() {
  const clientId = settingStore.settings.linuxdoClientId
  const redirectUri = encodeURIComponent(settingStore.settings.linuxdoCallbackUrl)
  window.location.href = `https://connect.linux.do/oauth2/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&response_type=code&scope=openid+profile+email`
}

linuxDoGetUser();
async function linuxDoGetUser() {
  const params = new URLSearchParams(window.location.search)
  const code = params.get('code')
  if (code) {
    oauthLoading.value = true
    oauthLinuxDoLogin(code).then(data => {
      bindForm.oauthUserId = data.userInfo.oauthUserId;
      if (!data.token) {
        showBindForm.value = true
        oauthLoading.value = false
        ElMessage({ message: '请注册绑定一个邮箱', type: 'warning', duration: 4000, plain: true })
        return;
      }
      saveToken(data.token);
    }).catch(() => { oauthLoading.value = false })
  }
  const cleanUrl = window.location.origin + window.location.pathname
  window.history.replaceState({}, '', cleanUrl)
}

function bind() {
  if (!bindForm.email) { ElMessage({ message: t('emptyEmailMsg'), type: 'error', plain: true }); return }
  if (getEmailName(bindForm.email).length < settingStore.settings.minEmailPrefix) {
    ElMessage({ message: t('minEmailPrefix', {msg: settingStore.settings.minEmailPrefix}), type: 'error', plain: true }); return
  }
  let email = getFullEmail(bindForm.email);
  if (!isEmail(email)) { ElMessage({ message: t('notEmailMsg'), type: 'error', plain: true }); return }
  if (settingStore.settings.regKey === 0 && !bindForm.code) { ElMessage({ message: t('emptyRegKeyMsg'), type: 'error', plain: true }); return }
  const f = {email, oauthUserId: bindForm.oauthUserId, code: bindForm.code}
  bindLoading.value = true
  oauthBindUser(f).then(data => { saveToken(data.token) }).catch(() => { bindLoading.value = false })
}

const submit = () => {
  if (!form.email) { ElMessage({ message: t('emptyEmailMsg'), type: 'error', plain: true }); return }
  let email = getFullEmail(form.email);
  if (!isEmail(email)) { ElMessage({ message: t('notEmailMsg'), type: 'error', plain: true }); return }
  if (!form.password) { ElMessage({ message: t('emptyPwdMsg'), type: 'error', plain: true }); return }
  loginLoading.value = true
  login(email, form.password).then(async data => { await saveToken(data.token) }).finally(() => { loginLoading.value = false })
}

async function saveToken(token) {
  localStorage.setItem('token', token)
  refreshWebsiteConfig()
  const user = await loginUserInfo();
  accountStore.currentAccountId = user.account.accountId;
  accountStore.currentAccount = user.account;
  userStore.user = user;
  const routers = permsToRouter(user.permKeys);
  routers.forEach(routerData => { router.addRoute('layout', routerData); });
  await router.replace({name: 'layout'})
  uiStore.showNotice()
  oauthLoading.value = false;
  bindLoading.value = false;
}

function refreshWebsiteConfig() {
  websiteConfig().then(setting => {
    settingStore.settings = setting
    settingStore.domainList = setting.domainList
    if (!suffix.value && setting.domainList.length > 0) { suffix.value = setting.domainList[0] }
    document.title = setting.title
  }).catch(e => { console.error(e) })
}

function submitRegister() {
  if (!registerForm.email) { ElMessage({ message: t('emptyEmailMsg'), type: 'error', plain: true }); return }
  if (getEmailName(registerForm.email).length < settingStore.settings.minEmailPrefix) {
    ElMessage({ message: t('minEmailPrefix', {msg: settingStore.settings.minEmailPrefix}), type: 'error', plain: true }); return
  }
  const email = getFullEmail(registerForm.email);
  if (!isEmail(email)) { ElMessage({ message: t('notEmailMsg'), type: 'error', plain: true }); return }
  if (!registerForm.password) { ElMessage({ message: t('emptyPwdMsg'), type: 'error', plain: true }); return }
  if (registerForm.password.length < 6) { ElMessage({ message: t('pwdLengthMsg'), type: 'error', plain: true }); return }
  if (registerForm.password !== registerForm.confirmPassword) { ElMessage({ message: t('confirmPwdFailMsg'), type: 'error', plain: true }); return }
  if (settingStore.settings.regKey === 0 && !registerForm.code) { ElMessage({ message: t('emptyRegKeyMsg'), type: 'error', plain: true }); return }

  if (!verifyToken && (settingStore.settings.registerVerify === 0 || (settingStore.settings.registerVerify === 2 && settingStore.settings.regVerifyOpen))) {
    if (!verifyShow.value) {
      verifyShow.value = true
      nextTick(() => {
        if (!turnstileId) {
          try { turnstileId = window.turnstile.render('.register-turnstile') }
          catch (e) { botJsError.value = true }
        } else { window.turnstile.reset('.register-turnstile') }
      })
    } else if (!botJsError.value) {
      ElMessage({ message: t('botVerifyMsg'), type: "error", plain: true })
    }
    return;
  }

  registerLoading.value = true
  const f = { email, password: registerForm.password, token: verifyToken, code: registerForm.code }
  register(f).then(({regVerifyOpen}) => {
    show.value = 'login'
    registerForm.email = ''; registerForm.password = ''; registerForm.confirmPassword = ''; registerForm.code = ''
    registerLoading.value = false; verifyToken = ''
    settingStore.settings.regVerifyOpen = regVerifyOpen
    verifyShow.value = false
    ElMessage({ message: t('regSuccessMsg'), type: 'success', plain: true })
  }).catch(res => {
    registerLoading.value = false
    if (res.code === 400) {
      verifyToken = ''; settingStore.settings.regVerifyOpen = true
      if (turnstileId) { window.turnstile.reset(turnstileId) }
      else { nextTick(() => { turnstileId = window.turnstile.render('.register-turnstile') }) }
      verifyShow.value = true
    }
  });
}
</script>

<!-- Global styles -->
<style>
.el-select-dropdown__item { padding: 0 15px; }
.no-autofill-pwd .el-input__inner { -webkit-text-security: disc !important; }
.gopk-dialog { border-radius: 14px !important; }
</style>

<style lang="scss" scoped>
$terracotta: #D97757;
$terracotta-dark: #C06642;
$parchment: #FAF8F5;
$parchment-dark: #F0EBE1;
$text-primary: #2D2A26;
$text-muted: #7A7570;
$border: #E2DDD5;

/* ══════════════════════
   Full-page wrapper
   ══════════════════════ */
#login-box {
  position: fixed;
  inset: 0;
  display: grid;
  grid-template-columns: 1fr 480px;
  background: $parchment;
  overflow: hidden;

  @media (max-width: 900px) {
    grid-template-columns: 1fr;
  }
}

/* ══════════════════════
   Ambient background
   ══════════════════════ */
.ambient-bg {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}

.ambient-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.55;
}

.orb-1 {
  width: 600px; height: 600px;
  background: radial-gradient(circle, rgba(217, 119, 87, 0.25) 0%, transparent 70%);
  top: -150px; left: -100px;
  animation: orb-float 18s ease-in-out infinite alternate;
}

.orb-2 {
  width: 500px; height: 500px;
  background: radial-gradient(circle, rgba(240, 180, 140, 0.2) 0%, transparent 70%);
  bottom: -120px; left: 200px;
  animation: orb-float 24s ease-in-out infinite alternate-reverse;
}

.orb-3 {
  width: 400px; height: 400px;
  background: radial-gradient(circle, rgba(200, 160, 120, 0.15) 0%, transparent 70%);
  top: 40%; left: 35%;
  animation: orb-float 20s ease-in-out infinite alternate;
}

@keyframes orb-float {
  0%   { transform: translate(0, 0) scale(1); }
  100% { transform: translate(40px, 30px) scale(1.08); }
}

/* Subtle grain texture */
.grain-overlay {
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E");
  opacity: 0.6;
}

/* ══════════════════════
   Brand / Left panel
   ══════════════════════ */
.brand-panel {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  padding: 60px 64px;

  @media (max-width: 900px) {
    display: none;
  }
}

.brand-content {
  max-width: 400px;
}

.brand-logo {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 40px;
}

.logo-icon {
  width: 48px; height: 48px;
  border-radius: 12px;
  background: linear-gradient(135deg, $terracotta 0%, $terracotta-dark 100%);
  color: #fff;
  display: flex; align-items: center; justify-content: center;
  box-shadow: 0 4px 16px rgba(217, 119, 87, 0.3);
  flex-shrink: 0;
}

.logo-text {
  font-family: 'Newsreader', Georgia, serif;
  font-size: 26px;
  font-weight: 400;
  letter-spacing: 0.01em;
  color: $text-primary;
}

.brand-tagline {
  font-family: 'Newsreader', Georgia, serif;
  font-size: 38px;
  font-weight: 400;
  line-height: 1.25;
  letter-spacing: -0.01em;
  color: $text-primary;
  margin-bottom: 40px;
}

.brand-features {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 14px;
  color: $text-muted;

  svg { color: $terracotta; flex-shrink: 0; }
}

/* ══════════════════════
   Form panel / Right
   ══════════════════════ */
.form-wrapper {
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.72);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-left: 1px solid rgba(226, 221, 213, 0.6);
  box-shadow: -4px 0 40px rgba(45, 42, 38, 0.06);

  @media (max-width: 900px) {
    background: $parchment;
    border-left: none;
    box-shadow: none;
    width: 100%;
  }
}

.container {
  width: 100%;
  max-width: 380px;
  padding: 48px 40px;

  @media (max-width: 480px) {
    padding: 32px 24px;
  }
}

/* Small logo (custom bg mode) */
.form-logo {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 28px;

  .logo-icon-sm {
    width: 32px; height: 32px;
    border-radius: 8px;
    background: linear-gradient(135deg, $terracotta 0%, $terracotta-dark 100%);
    color: #fff;
    display: flex; align-items: center; justify-content: center;
  }

  span {
    font-family: 'Newsreader', Georgia, serif;
    font-size: 18px;
    font-weight: 400;
    color: $text-primary;
  }
}

/* ── Form header ── */
.form-header {
  margin-bottom: 28px;
}

.form-title {
  font-family: 'Newsreader', Georgia, serif;
  font-size: 28px;
  font-weight: 400;
  letter-spacing: -0.01em;
  color: $text-primary;
  margin-bottom: 6px;
}

.form-desc {
  font-size: 13.5px;
  color: $text-muted;
  line-height: 1.5;
}

/* ── Fields ── */
.form-fields {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.field-group {
  margin-bottom: 14px;
}

.field-label {
  display: block;
  font-size: 12.5px;
  font-weight: 500;
  color: $text-muted;
  margin-bottom: 5px;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}

:deep(.el-input__wrapper) {
  border-radius: 8px !important;
  background: rgba(255,255,255,0.9) !important;
  box-shadow: 0 0 0 1px $border !important;
  transition: box-shadow 0.15s ease !important;
  height: 40px;

  &:hover { box-shadow: 0 0 0 1px darken($border, 8%) !important; }
  &.is-focus { box-shadow: 0 0 0 2px rgba(217, 119, 87, 0.35) !important; }
}

.email-input :deep(.el-input__wrapper) {
  border-radius: 8px 0 0 8px !important;
}

:deep(.el-input-group__append) {
  padding: 0 8px 0 6px !important;
  background: rgba(255,255,255,0.9) !important;
  border-radius: 0 8px 8px 0 !important;
  box-shadow: 0 0 0 1px $border !important;
  border-left: 1px solid $border !important;
}

:deep(.el-input__inner) {
  font-size: 14px !important;
  color: $text-primary !important;
}

.suffix-display {
  display: flex;
  align-items: center;
  gap: 2px;
  cursor: pointer;
  color: $text-muted;
  font-size: 13px;
  white-space: nowrap;
}

.el-input {
  width: 100%;
  :deep(.el-input__inner) { height: 38px; }
}

/* ── Buttons ── */
.btn {
  width: 100%;
  height: 40px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  letter-spacing: 0.01em;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
  margin: 0 !important;
}

.btn-primary {
  background: linear-gradient(135deg, $terracotta 0%, $terracotta-dark 100%) !important;
  color: #ffffff !important;
  box-shadow: 0 2px 8px rgba(217, 119, 87, 0.3) !important;
  margin-top: 8px !important;

  &:hover {
    box-shadow: 0 4px 16px rgba(217, 119, 87, 0.45) !important;
    transform: translateY(-1px);
  }

  &:active { transform: translateY(0); }
}

.btn-oauth {
  background: rgba(255,255,255,0.9) !important;
  color: $text-primary !important;
  border: 1px solid $border !important;
  box-shadow: 0 1px 4px rgba(45, 42, 38, 0.06) !important;

  &:hover {
    background: #fff !important;
    border-color: darken($border, 6%) !important;
  }
}

/* ── Divider ── */
.divider {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 14px 0;

  &::before, &::after {
    content: '';
    flex: 1;
    height: 1px;
    background: $border;
  }

  span {
    font-size: 12px;
    color: $text-muted;
    flex-shrink: 0;
  }
}

/* ── Switch link ── */
.switch {
  margin-top: 22px;
  text-align: center;
  font-size: 13.5px;
  color: $text-muted;

  span {
    color: $terracotta;
    cursor: pointer;
    font-weight: 500;
    margin-left: 4px;
    transition: opacity 0.15s;

    &:hover { opacity: 0.75; }
  }
}

/* ── GitHub link ── */
.github-link {
  position: fixed;
  bottom: 16px;
  right: 16px;
  z-index: 100;
  width: 34px; height: 34px;
  border-radius: 50%;
  background: rgba(255,255,255,0.9);
  border: 1px solid $border;
  box-shadow: 0 2px 8px rgba(45, 42, 38, 0.1);
  display: flex; align-items: center; justify-content: center;
  color: $text-muted;
  transition: all 0.2s ease;
  text-decoration: none;

  &:hover {
    color: $terracotta;
    border-color: rgba(217, 119, 87, 0.3);
    box-shadow: 0 3px 12px rgba(45, 42, 38, 0.12);
    transform: translateY(-1px);
  }
}

/* ── Bind dialog ── */
:deep(.bind-dialog) {
  width: 400px !important;
  @media (max-width: 440px) {
    width: calc(100% - 40px) !important;
    margin: 0 20px !important;
  }
}

.bind-container {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

/* ── Select (hidden) ── */
.select {
  position: absolute;
  right: 30px;
  width: 100px;
  opacity: 0;
  pointer-events: none;
}

.register-turnstile { margin-bottom: 14px; }

:deep(.el-button + .el-button) { margin: 0; }

/* ══════════════════════
   Dark mode
   ══════════════════════ */
.dark & {
  #login-box { background: #1a1915; }

  .form-wrapper {
    background: rgba(30, 29, 27, 0.85);
    border-left-color: rgba(58, 56, 53, 0.5);
  }

  .brand-tagline, .logo-text, .form-title { color: #EDE8DF; }
  .form-desc, .field-label, .feature-item { color: #8A8580; }

  :deep(.el-input__wrapper) {
    background: rgba(40, 38, 36, 0.9) !important;
    box-shadow: 0 0 0 1px #3A3835 !important;
    &.is-focus { box-shadow: 0 0 0 2px rgba(224, 139, 106, 0.35) !important; }
  }

  :deep(.el-input-group__append) {
    background: rgba(40, 38, 36, 0.9) !important;
    border-color: #3A3835 !important;
    box-shadow: 0 0 0 1px #3A3835 !important;
  }

  :deep(.el-input__inner) { color: #EDE8DF !important; }

  .btn-oauth {
    background: rgba(40, 38, 36, 0.9) !important;
    color: #EDE8DF !important;
    border-color: #3A3835 !important;
  }

  .divider::before, .divider::after { background: #3A3835; }

  .switch { color: #8A8580; }

  .github-link {
    background: rgba(40, 38, 36, 0.9);
    border-color: #3A3835;
  }
}
</style>
