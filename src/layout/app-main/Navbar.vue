<template>
  <div class="navbar rowBC reset-el-dropdown">
    <div class="rowSC">
      <!--  切换sidebar按钮  -->
      <hamburger
        v-if="settings.showHamburger"
        :is-active="sidebar.opened"
        class="hamburger-container"
        @toggleClick="toggleSideBar"
      />
      <!--  面包屑导航  -->
      <breadcrumb class="breadcrumb-container" />
    </div>
    <!--导航标题-->
    <div v-if="settings.showNavbarTitle" class="heardCenterTitle">{{ settings.title }}</div>
    <!-- 下拉操作菜单 -->
    <div v-if="settings.ShowDropDown" class="right-menu rowSC">
      <ScreenFull />
      <ScreenLock />
      <ThemeSelect />
      <SizeSelect />
      <LangSelect />
      <el-dropdown trigger="click" size="medium">
        <div class="avatar-wrapper">
          <img src="https://github.jzfai.top/file/images/nav-right-logo.gif" class="user-avatar" />
          <CaretBottom style="width: 1em; height: 1em; margin-left: 4px" />
        </div>
        <template #dropdown>
          <el-dropdown-menu>
            <router-link to="/">
              <el-dropdown-item>{{ langTitle('Home') }}</el-dropdown-item>
            </router-link>
            <a target="_blank" href="https://github.com/jzfai/vue3-admin-plus">
              <el-dropdown-item>{{ langTitle('Github') }}</el-dropdown-item>
            </a>
            <a target="_blank" href="https://github.jzfai.top/low-code-platform">
              <el-dropdown-item>{{ langTitle('low-code-platform') }}</el-dropdown-item>
            </a>
            <a target="_blank" href="https://github.jzfai.top/vue3-admin-doc">
              <el-dropdown-item>{{ langTitle('office-doc') }}</el-dropdown-item>
            </a>
            <!--<el-dropdown-item>修改密码</el-dropdown-item>-->
            <el-dropdown-item divided @click="loginOut">{{ langTitle('login out') }}</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>

<script setup lang="ts">
import { nextTick } from 'vue'
import { CaretBottom } from '@element-plus/icons-vue'
import { useRoute, useRouter } from 'vue-router'
import Breadcrumb from './Breadcrumb.vue'
import Hamburger from './Hamburger.vue'
import LangSelect from './component/LangSelect.vue'
import ScreenFull from './component/ScreenFull.vue'
import SizeSelect from './component/SizeSelect.vue'
import ThemeSelect from './component/ThemeSelect.vue'
import ScreenLock from './component/ScreenLock.vue'
import { resetState } from '@/hooks/use-permission'
import { elMessage } from '@/hooks/use-element'
import { useBasicStore } from '@/store/basic'
import { langTitle } from '@/hooks/use-common'

const basicStore = useBasicStore()
const { settings, sidebar, setToggleSideBar } = basicStore
const toggleSideBar = () => {
  setToggleSideBar()
}
//退出登录
const router = useRouter()
const route = useRoute()
const loginOut = () => {
  elMessage('退出登录成功')
  router.push(`/login?redirect=${route.path}`)
  nextTick(() => {
    resetState()
  })
}
</script>

<style lang="scss" scoped>
.navbar {
  height: var(--nav-bar-height);
  overflow: hidden;
  position: relative;
  background: var(--nav-bar-background);
  box-shadow: var(--nav-bar-box-shadow);
  z-index: 1;
}

//logo
.avatar-wrapper {
  margin-top: 5px;
  position: relative;
  cursor: pointer;

  .user-avatar {
    cursor: pointer;
    width: 40px;
    height: 40px;
    border-radius: 10px;
  }

  .el-icon-caret-bottom {
    cursor: pointer;
    position: absolute;
    right: -20px;
    top: 25px;
    font-size: 12px;
  }
}

//center-title
.heardCenterTitle {
  text-align: center;
  position: absolute;
  top: 50%;
  left: 46%;
  font-weight: 600;
  font-size: 20px;
  transform: translate(-50%, -50%);
}

//drop-down
.right-menu {
  cursor: pointer;
  margin-right: 40px;
  background-color: var(--nav-bar-right-menu-background);
}
</style>
