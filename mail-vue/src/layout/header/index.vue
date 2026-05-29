<template>
  <div class="header" :class="!hasPerm('email:send') ? 'not-send' : ''">
    <!-- Left: hamburger + breadcrumb -->
    <div class="header-btn">
      <hanburger @click="changeAside"></hanburger>
      <span class="breadcrumb-item">{{ $t(route.meta.title) }}</span>
    </div>

    <!-- Compose button -->
    <div v-perm="'email:send'" class="writer-box" @click="openSend">
      <div class="writer">
        <Icon icon="material-symbols:edit-outline-sharp" width="18" height="18"/>
        <span class="writer-label">{{ $t('compose') || 'Compose' }}</span>
      </div>
    </div>

    <!-- Toolbar: theme toggle, notice, user -->
    <div class="toolbar">
      <div v-if="uiStore.dark" class="sun-icon icon-item" @click="openDark($event)" title="Light mode">
        <Icon icon="mingcute:sun-fill" width="18" height="18"/>
      </div>
      <div v-else class="dark-icon icon-item" @click="openDark($event)" title="Dark mode">
        <Icon icon="solar:moon-linear" width="18" height="18"/>
      </div>

      <div class="notice icon-item" @click="openNotice" title="Announcements">
        <Icon icon="streamline-plump:announcement-megaphone" width="18" height="18"/>
      </div>

      <!-- User dropdown -->
      <el-dropdown ref="userinfoRef" @visible-change="e => userInfoShow = e" :teleported="false" popper-class="detail-dropdown gopk-dropdown">
        <div class="avatar" @click="userInfoHide">
          <div class="avatar-text">
            <div>{{ formatName(userStore.user.email) }}</div>
          </div>
          <Icon class="setting-icon" icon="mingcute:down-small-fill" width="18" height="18"/>
        </div>
        <template #dropdown>
          <div class="user-details">
            <div class="details-avatar">
              {{ formatName(userStore.user.email) }}
            </div>
            <div class="user-name">
              {{ userStore.user.name }}
            </div>
            <div class="detail-email" @click="copyEmail(userStore.user.email)">
              {{ userStore.user.email }}
            </div>
            <div class="detail-user-type">
              <el-tag class="gopk-tag">{{ userStore.user.role.name }}</el-tag>
            </div>
            <div class="action-info">
              <div>
                <span>{{ $t('sendCount') }}</span>
                <span>{{ $t('accountCount') }}</span>
              </div>
              <div>
                <div>
                  <span v-if="sendCount">{{ sendCount }}</span>
                  <el-tag v-if="!hasPerm('email:send')" class="gopk-tag">{{ sendType }}</el-tag>
                  <el-tag v-else class="gopk-tag">{{ sendType }}</el-tag>
                </div>
                <div>
                  <el-tag v-if="settingStore.settings.manyEmail || settingStore.settings.addEmail" class="gopk-tag">
                    {{ $t('disabled') }}
                  </el-tag>
                  <span v-else-if="accountCount && hasPerm('account:add')">{{ $t('totalUserAccount', {msg: accountCount}) }}</span>
                  <el-tag v-else-if="!accountCount && hasPerm('account:add')" class="gopk-tag">{{ $t('unlimited') }}</el-tag>
                  <el-tag v-else-if="!hasPerm('account:add')" class="gopk-tag">{{ $t('unauthorized') }}</el-tag>
                </div>
              </div>
            </div>
            <div class="logout">
              <el-button class="gopk-logout-btn" :loading="logoutLoading" @click="clickLogout">
                {{ $t('logOut') }}
              </el-button>
            </div>
          </div>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>

<script setup>
import router from "@/router";
import hanburger from '@/components/hamburger/index.vue'
import {logout} from "@/request/login.js";
import {Icon} from "@iconify/vue";
import {useUiStore} from "@/store/ui.js";
import {useUserStore} from "@/store/user.js";
import {useRoute} from "vue-router";
import {computed, ref} from "vue";
import {useSettingStore} from "@/store/setting.js";
import {hasPerm} from "@/perm/perm.js"
import {useI18n} from "vue-i18n";
import {setExtend} from "@/utils/day.js"

const {t} = useI18n();
const route = useRoute();
const settingStore = useSettingStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const logoutLoading = ref(false)
const userInfoShow = ref(false)
const userinfoRef = ref({})

const accountCount = computed(() => userStore.user.role.accountCount)

const sendType = computed(() => {
  if (settingStore.settings.send === 1) return t('disabled')
  if (!hasPerm('email:send')) return t('unauthorized')
  if (userStore.user.role.sendType === 'ban') return t('sendBanned')
  if (userStore.user.role.sendType === 'internal') return t('sendInternal')
  if (!userStore.user.role.sendCount) return t('unlimited')
  if (userStore.user.role.sendType === 'day') return t('daily')
  if (userStore.user.role.sendType === 'count') return t('total')
})

const sendCount = computed(() => {
  if (!hasPerm('email:send')) return null
  if (userStore.user.role.sendType === 'ban') return null
  if (userStore.user.role.sendType === 'internal') return null
  if (!userStore.user.role.sendCount) return null
  if (settingStore.settings.send === 1) return null
  return userStore.user.sendCount + '/' + userStore.user.role.sendCount
})

function userInfoHide() {
  if (userInfoShow.value) {
    userinfoRef.value.handleClose()
  } else {
    userinfoRef.value.handleOpen()
  }
}

async function copyEmail(email) {
  try {
    await navigator.clipboard.writeText(email);
    ElMessage({ message: t('copySuccessMsg'), type: 'success', plain: true })
  } catch (err) {
    ElMessage({ message: t('copyFailMsg'), type: 'error', plain: true })
  }
}

function openNotice() {
  uiStore.showNotice()
}

function openDark(e) {
  const nextIsDark = !uiStore.dark
  const root = document.documentElement
  if (!document.startViewTransition) {
    switchDark(nextIsDark, root)
    return
  }
  const x = e.clientX
  const y = e.clientY
  const maxX = Math.max(x, window.innerWidth - x)
  const maxY = Math.max(y, window.innerHeight - y)
  const endRadius = Math.hypot(maxX, maxY)
  root.setAttribute('data-theme-to', nextIsDark ? 'dark' : 'light')
  root.style.setProperty('--vt-x', `${x}px`)
  root.style.setProperty('--vt-y', `${y}px`)
  root.style.setProperty('--vt-end-radius', `${endRadius + 10}px`)
  const transition = document.startViewTransition(() => { switchDark(nextIsDark, root) })
  transition.finished.finally(() => { root.removeAttribute('data-theme-to') })
}

function switchDark(nextIsDark, root) {
  root.setAttribute('class', nextIsDark ? 'dark' : '')
  const metaTag = document.getElementById('theme-color-meta')
  const isMobile = !window.matchMedia("(pointer: fine) and (hover: hover)").matches
  metaTag.setAttribute('content', nextIsDark
    ? (isMobile ? '#1a1915' : '#111110')
    : (isMobile ? '#FAF8F5' : '#F5F0E8'))
  uiStore.dark = nextIsDark
}

function openSend() {
  uiStore.writerRef.open()
}

function changeAside() {
  uiStore.asideShow = !uiStore.asideShow
}

function clickLogout() {
  logoutLoading.value = true
  logout().then(() => {
    localStorage.removeItem("token")
    router.replace('/login')
  }).finally(() => { logoutLoading.value = false })
}

function formatName(email) {
  return email[0]?.toUpperCase() || ''
}
</script>

<!-- Global dropdown styles (not scoped) -->
<style>
.gopk-dropdown.detail-dropdown {
  color: var(--el-text-color-primary) !important;
  border-radius: 12px !important;
  border: 1px solid rgba(217, 119, 87, 0.15) !important;
  box-shadow: 0 8px 32px rgba(45, 42, 38, 0.12) !important;
  overflow: hidden;
}
</style>

<style lang="scss" scoped>
$terracotta: #D97757;
$terracotta-light: rgba(217, 119, 87, 0.1);
$text-primary: #2D2A26;
$text-muted: #7A7570;
$border-color: rgba(45, 42, 38, 0.1);

/* ── Header shell ── */
.header {
  display: grid;
  height: 100%;
  gap: 10px;
  grid-template-columns: auto auto 1fr;
  align-items: center;
  padding: 0 16px 0 4px;
  background: var(--el-bg-color);
}

.header.not-send {
  grid-template-columns: auto 1fr;
}

/* ── Left section ── */
.header-btn {
  display: inline-flex;
  align-items: center;
  height: 100%;
  min-width: 0;
  gap: 4px;
}

.breadcrumb-item {
  font-family: 'Newsreader', Georgia, serif;
  font-weight: 400;
  font-size: 16px;
  letter-spacing: 0.01em;
  color: var(--el-text-color-primary);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

/* ── Compose button ── */
.writer-box {
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;

  .writer {
    display: flex;
    align-items: center;
    gap: 7px;
    height: 32px;
    padding: 0 14px;
    border-radius: 8px;
    background: linear-gradient(135deg, $terracotta 0%, darken($terracotta, 8%) 100%);
    color: #ffffff;
    font-size: 13px;
    font-weight: 500;
    letter-spacing: 0.01em;
    transition: all 0.2s ease;
    box-shadow: 0 1px 6px rgba(217, 119, 87, 0.3);

    .writer-label {
      @media (max-width: 500px) {
        display: none;
      }
    }
  }

  &:hover .writer {
    box-shadow: 0 3px 12px rgba(217, 119, 87, 0.45);
    transform: translateY(-1px);
  }

  &:active .writer {
    transform: translateY(0);
    box-shadow: 0 1px 4px rgba(217, 119, 87, 0.3);
  }
}

/* ── Toolbar ── */
.toolbar {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 4px;

  @media (max-width: 767px) { gap: 2px; }

  .icon-item {
    width: 32px;
    height: 32px;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    color: $text-muted;
    transition: background 0.15s ease, color 0.15s ease;
  }

  .icon-item:hover {
    background: rgba(217, 119, 87, 0.08);
    color: $terracotta;
  }

  .notice {
    font-size: 18px;
  }

  /* ── Avatar ── */
  .avatar {
    display: flex;
    align-items: center;
    gap: 2px;
    cursor: pointer;
    padding: 2px 6px 2px 2px;
    border-radius: 8px;
    transition: background 0.15s ease;

    &:hover {
      background: rgba(217, 119, 87, 0.08);
    }

    .avatar-text {
      width: 30px;
      height: 30px;
      border-radius: 8px;
      background: linear-gradient(135deg, $terracotta 0%, darken($terracotta, 8%) 100%);
      color: #ffffff;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 13px;
      font-weight: 600;
      letter-spacing: 0.02em;
    }

    .setting-icon {
      color: $text-muted;
    }
  }
}

/* ── User detail dropdown ── */
:deep(.el-popper.is-pure) {
  border-radius: 12px;
}

.user-details {
  width: 260px;
  font-size: 13.5px;
  display: grid;
  grid-template-columns: 1fr;
  justify-items: center;
  padding-bottom: 4px;

  .details-avatar {
    margin-top: 20px;
    height: 44px;
    width: 44px;
    background: linear-gradient(135deg, $terracotta 0%, darken($terracotta, 8%) 100%);
    color: #ffffff;
    font-size: 20px;
    font-weight: 600;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 12px;
    box-shadow: 0 3px 12px rgba(217, 119, 87, 0.3);
  }

  .user-name {
    font-weight: 600;
    margin-top: 10px;
    padding: 0 20px;
    width: 260px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    text-align: center;
    font-size: 14px;
    color: var(--el-text-color-primary);
  }

  .detail-email {
    padding: 0 20px;
    margin-top: 3px;
    width: 260px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    text-align: center;
    color: $text-muted;
    font-size: 12.5px;
    cursor: pointer;
    transition: color 0.15s;

    &:hover { color: $terracotta; }
  }

  .detail-user-type {
    margin-top: 10px;
  }

  .gopk-tag {
    border-radius: 6px;
    border-color: rgba(217, 119, 87, 0.3);
    color: $terracotta;
    background: rgba(217, 119, 87, 0.08);
  }

  .action-info {
    width: 100%;
    display: grid;
    grid-template-columns: auto auto;
    margin-top: 14px;
    padding: 10px 16px;
    background: var(--el-fill-color-light);
    border-radius: 8px;
    margin-left: 10px;
    margin-right: 10px;
    width: calc(100% - 20px);
    gap: 6px;

    > div:first-child {
      display: grid;
      align-items: center;
      gap: 8px;
      color: $text-muted;
      font-size: 12px;
    }

    > div:last-child {
      display: grid;
      gap: 8px;
      text-align: right;

      > div {
        display: flex;
        align-items: center;
        justify-content: flex-end;
      }
    }
  }

  .logout {
    margin-top: 14px;
    width: 100%;
    padding: 0 12px 8px;

    .gopk-logout-btn {
      width: 100%;
      border-radius: 8px;
      height: 32px;
      font-size: 13px;
      background: linear-gradient(135deg, $terracotta 0%, darken($terracotta, 8%) 100%);
      border: none;
      color: #ffffff;
      font-weight: 500;
      transition: all 0.2s ease;

      &:hover {
        box-shadow: 0 3px 12px rgba(217, 119, 87, 0.4);
        opacity: 0.92;
      }
    }
  }
}

.el-tooltip__trigger:first-child:focus-visible {
  outline: unset;
}
</style>
