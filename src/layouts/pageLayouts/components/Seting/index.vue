<template>
  <el-drawer v-model="drawer" title="设置" @close="emit('update:modelValue', false)">
    <div class="drawer-content">
      <div class="layout_seting">
        <div class="sidebar_seting">
          <el-tooltip
            v-for="item in sidebarSeting"
            :key="item.value"
            :content="item.label"
            placement="bottom"
          >
            <div
              class="sidebar_mode"
              :class="{ 'sidebar_mode-select': appConfigMode.sidebarMode === item.value }"
              @click="handerShowElmenu(item.value)"
            >
              <div></div>
              <div></div>
            </div>
          </el-tooltip>
        </div>
      </div>
    </div>
  </el-drawer>
</template>

<script setup lang="ts">
  import { getAppCollapseMenu } from '@/hooks/userAppWindow';
  import { watch, ref } from 'vue';
  import { useAppStoreHook } from '@/store/modules/app';
  import { SidebarMode } from '@/store/types';

  const props = defineProps({
    modelValue: {
      type: Boolean,
      default: false,
    },
  });

  const emit = defineEmits(['update:modelValue']);

  const drawer = ref<boolean>(props.modelValue);
  watch(
    () => props.modelValue,
    (count: boolean) => {
      drawer.value = count;
      /* ... */
    },
  );

  const { appConfigMode } = getAppCollapseMenu();
  const isAppConfigMode = ref(appConfigMode.value);
  const sidebarSeting: { label: string; value: SidebarMode }[] = [
    {
      label: '左侧菜单模式',
      value: 'vertical',
    },
    {
      label: '顶部菜单模式',
      value: 'horizontal',
    },
    {
      label: '混合菜单模式',
      value: 'blend',
    },
  ];

  const appStore = useAppStoreHook();
  // 折叠菜单事件
  const handerShowElmenu = (vale: SidebarMode) => {
    isAppConfigMode.value.sidebarMode = vale;
    appStore.setAppConfigMode(isAppConfigMode.value);
  };
</script>

<style lang="scss" scoped>
  .drawer-content {
    width: 100%;
    height: 100%;

    .layout_seting {
      .sidebar_seting {
        display: flex;
        align-items: center;
        justify-content: space-around;

        .sidebar_mode-select {
          border: 2px solid #409eff;
        }

        .sidebar_mode {
          position: relative;
          width: 115px;
          height: 80px;
          background: #f0f2f5;
          border-radius: 5px;
          box-shadow: 0 1px 2.5px 0 rgb(0 0 0 / 18%);

          &:nth-child(1) {
            div {
              &:nth-child(1) {
                width: 30%;
                height: 100%;
                background: #409eff;
              }

              &:nth-child(2) {
                position: absolute;
                top: 0;
                right: 0;
                width: 70%;
                height: 30%;
                background: #fff;
                box-shadow: 0 0 1px #888;
              }
            }
          }

          &:nth-child(2) {
            div {
              &:nth-child(1) {
                width: 100%;
                height: 30%;
                background: #409eff;
                box-shadow: 0 0 1px #888;
              }
            }
          }

          &:nth-child(3) {
            div {
              &:nth-child(1) {
                width: 30%;
                height: 100%;
                background: #fff;
              }

              &:nth-child(2) {
                position: absolute;
                top: 0;
                right: 0;
                width: 100%;
                height: 30%;
                background: #409eff;
                box-shadow: 0 0 1px #888;
              }
            }
          }
        }
      }
    }
  }
</style>
