<template>
  <div class="theme" :class="{ 'theme-dark': color === 'dark' }" @click="toggleDarkMode">
    <div class="theme-inner"></div>
    <SvgIcon name="sun"></SvgIcon>
    <SvgIcon name="moon"></SvgIcon>
  </div>
</template>

<script setup lang="ts">
  import SvgIcon from '../SvgIcon/index.vue';
  import { useColorMode } from '@vueuse/core';
  import { useAppStoreHook } from '@/store/modules/app';

  const appStore = useAppStoreHook();
  const color = useColorMode();
  const toggleDarkMode = () => {
    color.value = color.value === 'dark' ? 'light' : 'dark';
    appStore.appConfigMode.themeMode = color.value;
    appStore.setAppConfigMode(appStore.appConfigMode);
  };
</script>

<style lang="scss" scoped>
  .theme {
    position: relative;
    width: 50px;
    height: 26px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: 151515;
    border: 1px solid var(--text-color-primary);
    padding: 0 6px;
    font-size: 1em;
    border-radius: 30px;
    .theme-inner {
      position: absolute;
      z-index: 1;
      width: 18px;
      height: 18px;
      background-color: var(--text-color-primary);
      border-radius: 50%;
      transition: transform 0.5s, background-color 0.5s;
      will-change: transform;
    }
  }
  .theme-dark {
    .theme-inner {
      transform: translateX(calc(100% + 2px));
    }
  }
  .icon {
    font-size: 1em;
  }
</style>
