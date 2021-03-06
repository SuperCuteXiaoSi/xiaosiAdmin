<template>
  <div class="main-container-tabs">
    <el-tabs
      v-model="editableTabsValue"
      type="card"
      class="tabs-view"
      :closable="multiTabs.length > 1"
      @tab-remove="tabRemoveChange"
    >
      <el-tab-pane v-for="item in multiTabs" :key="setTabPaneKey(item)" :name="setTabPaneKey(item)">
        <template #label>
          <div
            class="tabs-view-item"
            @click="changeTab(item)"
            @contextmenu.prevent="contextmenu(item.path, $event)"
          ></div>
          <span>{{ translateI18n(item.meta.title) }}</span>
        </template>
      </el-tab-pane>
    </el-tabs>
    <transition name="el-zoom-in-top">
      <ul v-show="visible" class="right-view" :style="rightViewStyle">
        <div
          v-for="(item, key) in rightClickTags"
          :key="key"
          class="right-view-item cursor"
          :class="{ disabled: item.disabled }"
          @click="rightViewChange(item)"
        >
          <li>
            <span>{{ item.text }}</span>
          </li>
        </div>
      </ul>
    </transition>
    <div class="right-button">
      <ul>
        <li class="cursor" @click="onFresh()">
          <svg-icon class="rotate" name="iEL-refresh"></svg-icon>
        </li>
        <li>
          <el-dropdown trigger="click" placement="bottom-end">
            <svg-icon class="action-item" name="iEL-arrow-down"></svg-icon>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item
                  v-for="(item, key) in rightClickTags"
                  :key="key"
                  :command="{ key, item }"
                  :disabled="item.disabled"
                  @click="rightViewChange(item)"
                >
                  {{ item.text }}
                </el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { translateI18n } from '@/hooks/web/useI18n';
  import { usePermissionStoreHook } from '@/store/modules/permission';
  import type { MultiTabsType } from '@/store/types';
  import { ref, computed, watch, onBeforeMount } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import SvgIcon from '@/components/SvgIcon/index.vue';
  import { useTabsView } from './hooks/useTabsView';
  import { useTabsChange } from './hooks/useTabsChange';
  import { emitter } from '@/utils/mitt';
  import { useNavSideBar } from '../../hooks/useNavSideBar';

  const route = useRoute();
  const router = useRouter();

  const multiTabs = computed<MultiTabsType[]>(() => usePermissionStoreHook().multiTabs);

  const { setTabPaneKey, addRouteTabs, onFresh, removeTab } = useTabsChange(multiTabs);

  const { selectMenu } = useNavSideBar();

  const { visible, rightClickTags, rightViewStyle, contextmenu, rightViewChange } =
    useTabsView(multiTabs);

  const editableTabsValue = ref(setTabPaneKey(route));

  watch(
    () => route.path,
    () => {
      editableTabsValue.value = setTabPaneKey(route);
    },
  );

  onBeforeMount(() => {
    emitter.on('siteBarChange', ({ routeRaw }) => {
      addRouteTabs(routeRaw as unknown as MultiTabsType);
      contextmenu(routeRaw.path);
    });

    selectMenu(route.path);
    contextmenu(route.path);
  });

  const tabRemoveChange = (e: string) => {
    removeTab(e);
    setTimeout(() => {
      contextmenu(route.path);
    });
  };

  const changeTab = (e: MultiTabsType) => {
    router.push({
      path: e.path,
      query: e.query,
    });
    contextmenu(e.path);
  };
</script>

<style lang="scss" scoped>
  .main-container-tabs {
    display: flex;
    position: relative;

    // .el-tabs :deep(.el-tabs__header) {
    //   margin: 0;
    // }
    .tabs-view {
      .tabs-view-item {
        width: 100%;
        height: 100%;
        position: absolute;
        left: 0;
      }
    }
    .el-tabs {
      width: 0;
      height: $tabsPageHeight;
      flex: 1;
      margin: 0 10px;
      :deep(.el-tabs__header) {
        height: 100%;
        margin: 0;
        border: none;
        // display: flex;
        // align-items: center;
        .el-tabs__nav-wrap,
        .el-tabs__nav-scroll {
          height: 100%;
        }

        .el-tabs__nav-scroll {
          display: flex;
          align-items: center;
          .el-tabs__nav {
            border: none;
            border-radius: 0;
            .el-tabs__item {
              height: 100%;
              line-height: $tabsPageHeight - 10;
              border-radius: 4px;
              border: 1px solid var(--border-color-light);
              margin-right: 4px;
            }
          }
        }

        .el-tabs__nav-next,
        .el-tabs__nav-prev {
          height: $tabsPageHeight;
          line-height: $tabsPageHeight;
        }
        // .el-tabs__nav-*{

        // }
      }
    }
    .right-view {
      position: fixed;
      z-index: 999;
      background-color: var(--main-bg-color);
      border: 1px solid var(--border-color-light);
      border-radius: 5px;
      padding: 5px;
      box-shadow: 0px 0px 12px rgb(28 29 30 / 8%);
      .right-view-item {
        padding: 5px 10px;
        font-size: 14px;
        display: flex;
        align-items: center;
        border-radius: 5px;
        .right-view-item-icon {
          margin-right: 5px;
        }
        &:hover {
          background-color: var(--sub-color-2);
        }
      }
    }

    .right-button {
      ul {
        display: flex;
        li {
          width: $tabsPageHeight;
          height: 100%;
          text-align: center;
          line-height: $tabsPageHeight;
          font-size: 16px;
          border: 1px solid var(--border-color-light);
          .action-item {
            height: $tabsPageHeight;
          }
        }
        /* 刷新按钮动画效果 */
        .refresh-button {
          animation: rotate 600ms linear infinite;
        }
        @keyframes rotate {
          from {
            transform: rotate(0deg);
          }

          to {
            transform: rotate(-360deg);
          }
        }
      }
    }
  }
</style>
