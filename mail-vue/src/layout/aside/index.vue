<template>
  <el-scrollbar class="scroll">
    <div>
      <!-- GOPK Mail Logo / Title -->
      <div class="title">
        <div class="logo-mark">
          <Icon icon="mdi:email-outline" width="18" height="18" />
        </div>
        <div class="title-text">GOPK Mail</div>
      </div>

      <el-menu :collapse="false" style="margin-top: 8px">
        <el-menu-item @click="router.push({name: 'email'})" index="email"
                      :class="route.meta.name === 'email' ? 'choose-item' : ''">
          <Icon icon="hugeicons:mailbox-01" width="18" height="18" />
          <span class="menu-name">{{$t('inbox')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'send'})" index="send" v-perm="'email:send'"
                      :class="route.meta.name === 'send' ? 'choose-item' : ''">
          <Icon icon="cil:send" width="18" height="18" />
          <span class="menu-name">{{$t('sent')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'draft'})" index="draft" v-perm="'email:send'"
                      :class="route.meta.name === 'draft' ? 'choose-item' : ''">
          <Icon icon="ep:document" width="18" height="18" />
          <span class="menu-name">{{$t('drafts')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'star'})" index="star"
                      :class="route.meta.name === 'star' ? 'choose-item' : ''">
          <Icon icon="solar:star-line-duotone" width="18" height="18" />
          <span class="menu-name">{{$t('starred')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'setting'})" index="setting"
                      :class="route.meta.name === 'setting' ? 'choose-item' : ''">
          <Icon icon="fluent:settings-48-regular" width="18" height="18" />
          <span class="menu-name">{{$t('settings')}}</span>
        </el-menu-item>

        <!-- Manage section -->
        <div class="manage-title" v-perm="['all-email:query','user:query','role:query','setting:query','analysis:query','reg-key:query']">
          <span>{{$t('manage')}}</span>
        </div>
        <el-menu-item @click="router.push({name: 'analysis'})" index="analysis" v-perm="'analysis:query'"
                      :class="route.meta.name === 'analysis' ? 'choose-item' : ''">
          <Icon icon="fluent:data-pie-20-regular" width="18" height="18" />
          <span class="menu-name">{{$t('analytics')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'user'})" index="setting" v-perm="'user:query'"
                      :class="route.meta.name === 'user' ? 'choose-item' : ''">
          <Icon icon="si:user-alt-2-line" width="18" height="18" />
          <span class="menu-name">{{$t('allUsers')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'all-email'})" index="all-email" v-perm="'all-email:query'"
                      :class="route.meta.name === 'all-email' ? 'choose-item' : ''">
          <Icon icon="fluent:mail-list-28-regular" width="18" height="18" />
          <span class="menu-name">{{$t('allMail')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'role'})" index="setting" v-perm="'role:query'"
                      :class="route.meta.name === 'role' ? 'choose-item' : ''">
          <Icon icon="fluent:lock-closed-16-regular" width="18" height="18" />
          <span class="menu-name">{{$t('permissions')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'reg-key'})" index="reg-key" v-perm="'reg-key:query'"
                      :class="route.meta.name === 'reg-key' ? 'choose-item' : ''">
          <Icon icon="fluent:fingerprint-20-filled" width="18" height="18" />
          <span class="menu-name">{{$t('inviteCode')}}</span>
        </el-menu-item>
        <el-menu-item @click="router.push({name: 'sys-setting'})" index="sys-setting" v-perm="'setting:query'"
                      :class="route.meta.name === 'sys-setting' ? 'choose-item' : ''">
          <Icon icon="eos-icons:system-ok-outlined" width="18" height="18" />
          <span class="menu-name">{{$t('SystemSettings')}}</span>
        </el-menu-item>
      </el-menu>
    </div>
  </el-scrollbar>
</template>

<script setup>
import router from "@/router/index.js";
import { useRoute } from "vue-router";
import {Icon} from "@iconify/vue";
import {useSettingStore} from "@/store/setting.js";

const settingStore = useSettingStore();
const route = useRoute();
</script>

<style lang="scss" scoped>

/* ── Claude-style palette variables ── */
/* Light: warm parchment / linen background */
/* Dark: deep charcoal with warm undertones */
$aside-bg-light: #F5F0E8;
$aside-bg-dark: #1E1D1B;
$terracotta: #D97757;
$terracotta-hover: #C96A45;
$text-light: #2D2A26;
$text-muted-light: #7A7570;
$text-dark: #EDE8DF;
$text-muted-dark: #8A8580;

.scroll {
  height: 100%;
}

:deep(.el-scrollbar__wrap--hidden-default) {
  background: $aside-bg-light !important;
}

:deep(.el-menu) {
  background: transparent;
  border-right: none;
}

:deep(.el-menu-item) {
  background: transparent;
  color: $text-light;
}

/* Dark mode overrides via CSS vars */
.dark :deep(.el-scrollbar__wrap--hidden-default) {
  background: $aside-bg-dark !important;
}

.dark :deep(.el-menu-item) {
  color: $text-dark;
}

/* ── Title / Logo ── */
.title {
  margin: 16px 12px 4px;
  height: 48px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 14px;
  background: linear-gradient(135deg, $terracotta 0%, $terracotta-hover 100%);
  box-shadow: 0 2px 12px rgba(217, 119, 87, 0.28);
  max-width: 236px;
  transition: box-shadow 0.2s ease;

  &:hover {
    box-shadow: 0 4px 16px rgba(217, 119, 87, 0.38);
  }
}

.logo-mark {
  color: rgba(255, 255, 255, 0.92);
  display: flex;
  align-items: center;
  flex-shrink: 0;
}

.title-text {
  font-family: 'Newsreader', 'Georgia', serif;
  font-size: 17px;
  font-weight: 400;
  letter-spacing: 0.01em;
  color: rgba(255, 255, 255, 0.95);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  max-width: 180px;
}

/* ── Section divider ── */
.manage-title {
  margin: 18px 0 6px;
  padding: 0 22px;
  font-size: 10.5px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: $text-muted-light;
  user-select: none;
}

/* ── Menu items ── */
.el-menu-item {
  margin: 2px 10px !important;
  border-radius: 8px !important;
  height: 36px !important;
  padding: 0 12px !important;
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 13.5px;
  font-weight: 450;
  color: $text-light;
  transition: background 0.15s ease, color 0.15s ease !important;
}

.menu-name {
  user-select: none;
  margin-left: 0 !important;
}

/* Active item */
.choose-item {
  background: rgba(217, 119, 87, 0.12) !important;
  color: $terracotta !important;
  font-weight: 550 !important;

  .menu-name {
    color: $terracotta;
  }
}

/* Hover */
@media (hover: hover) {
  .el-menu-item:not(.choose-item):hover {
    background: rgba(45, 42, 38, 0.06) !important;
    color: $text-light !important;
  }
}

/* ── Dark mode ── */
:deep(.dark) {
  .el-scrollbar__wrap--hidden-default {
    background: $aside-bg-dark !important;
  }
  .el-menu-item {
    color: $text-dark;
  }
  .manage-title {
    color: $text-muted-dark;
  }
}

/* Use CSS var fallback for dark mode since scoped can't reach html.dark easily */
/* We rely on the parent providing --aside-background override */
:deep(.el-scrollbar__wrap--hidden-default) {
  background: var(--aside-background, $aside-bg-light) !important;
}

.el-menu {
  border-right: 0;
  width: 260px;
  background: transparent !important;
}
</style>
