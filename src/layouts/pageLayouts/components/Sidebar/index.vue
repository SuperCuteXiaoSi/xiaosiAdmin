<template>
  <el-scrollbar wrap-class="scrollbar-wrapper">
    <el-menu
      :default-active="activeMenyu"
      :unique-opened="true"
      :collapse="appConfigMode.sidebarMode === 'horizontal' ? false : appConfigMode.collapseMenu"
      :mode="mode"
      @select="(indexPath) => selectMenu(indexPath)"
    >
      <sidebar-item
        v-for="menuRoute in menuData"
        :key="menuRoute.path"
        :item="menuRoute"
        :is-nest="false"
        :base-path="menuRoute.path"
      />
    </el-menu>
  </el-scrollbar>
</template>

<script setup lang="ts">
  import { computed, PropType, ref, watch } from 'vue';
  import { useRoute } from 'vue-router';
  import SidebarItem from './SidebarItem.vue';
  import { getAppCollapseMenu } from '@/hooks/userAppWindow';
  import { usePermissionStoreHook } from '@/store/modules/permission';
  import { AppRouteRecordRaw } from '#/route';
  import { getParentPaths, findRouteByPath } from '@/router/utils';
  import { useNavSideBar } from '../../hooks/useNavSideBar';

  const { selectMenu } = useNavSideBar();

  const route = useRoute();
  const { appConfigMode } = getAppCollapseMenu();
  defineProps({
    mode: {
      type: String as PropType<'vertical' | 'horizontal'>,
      default: 'vertical',
    },
  });

  let subMenuData = ref<AppRouteRecordRaw[]>(usePermissionStoreHook().wholeMenus);

  const menuData = computed<AppRouteRecordRaw[]>(() => {
    return appConfigMode.value.sidebarMode === 'blend'
      ? subMenuData.value
      : usePermissionStoreHook().wholeMenus;
  });

  function getSubMenuData(path: string) {
    // path的父级路由组成的数组
    const parentPathArr = getParentPaths(path, usePermissionStoreHook().wholeMenus);
    // 当前路由的信息
    const parenetRoute = findRouteByPath(parentPathArr[0], usePermissionStoreHook().wholeMenus);
    if (parenetRoute) {
      if (parenetRoute.children && !parenetRoute.children[0].hidden)
        subMenuData.value = parenetRoute.children;
      else subMenuData.value = [parenetRoute];
    }
  }

  getSubMenuData(route.path);
  watch(
    () => [route.path, appConfigMode.value.sidebarMode],
    () => {
      if (appConfigMode.value.sidebarMode === 'blend') {
        getSubMenuData(route.path);
      }
    },
  );

  const activeMenyu = computed<string>(() => {
    const { meta, path } = route;
    if (meta.activeMenu) {
      return meta.activeMenu as string;
    }
    return path;
  });

  // const isCollapse = ref(false)
</script>

<style lang="scss" scoped></style>
