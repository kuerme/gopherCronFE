<template>
  <q-layout
    view="hHh Lpr lff"
    class="rounded-borders tw-flex tw-flex-no-wrap tw-h-full tw-w-full"
  >
    <q-header class="bg-black">
      <q-toolbar class="tw-gap-4">
        <q-btn flat round dense icon="menu" @click="drawer = !drawer" />

        <span
          class="tw-text-lg tw-pl-2 tw-tracking-wide tw-font-bold tw-cursor-pointer"
          >作业调度系统</span
        >
        <span v-if="version" class="tw-text-sm">{{ version }}</span>
        <div class="tw-hidden md:tw-flex"><org-select></org-select></div>

        <q-space />

        <q-btn
          flat
          round
          dense
          :icon="
            notificationSetting.status ? 'notifications' : 'notifications_off'
          "
          @click="switchNotificationSetting"
        >
          <q-tooltip>
            {{
              notificationSetting.status
                ? '点击关闭任务状态通知'
                : '点击开启任务状态通知'
            }}
          </q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="drawer"
      :mini="miniState"
      :width="200"
      :breakpoint="$q.screen.sizes.md"
    >
      <q-scroll-area style="margin-bottom: 80px; height: calc(100% - 80px)">
        <q-list padding>
          <q-item v-show="showMenuOrgSelector" class="tw-flex md:tw-hidden">
            <org-select behavior="dialog"></org-select>
          </q-item>
          <q-item
            clickable
            :to="{
              name: 'summary',
              params: { orgId: store.getters.currentOrg },
            }"
          >
            <q-item-section avatar>
              <q-icon name="timeline" />
            </q-item-section>

            <q-item-section class="tw-font-medium">看板</q-item-section>
          </q-item>

          <q-item
            clickable
            :to="{
              name: 'projects',
              params: { orgId: store.getters.currentOrg },
            }"
          >
            <q-item-section avatar>
              <q-icon name="view_list" />
            </q-item-section>

            <q-item-section class="tw-font-medium">项目管理</q-item-section>
          </q-item>

          <q-item
            clickable
            :to="{
              name: 'workflows',
              params: { orgId: store.getters.currentOrg },
            }"
          >
            <q-item-section avatar>
              <q-icon name="mediation" />
            </q-item-section>

            <q-item-section class="tw-font-medium">任务编排</q-item-section>
          </q-item>

          <q-separator class="tw-bg-stone-800" />

          <q-expansion-item
            v-if="store.getters.isAdmin"
            v-model="adminMenuExpanded"
            label="系统管理"
            header-class="tw-font-medium"
            icon="admin_panel_settings"
          >
            <q-item
              clickable
              :inset-level="0.2"
              :to="{
                name: 'user-admin',
                params: { orgId: store.getters.currentOrg },
              }"
            >
              <q-item-section avatar>
                <q-icon name="supervisor_account" />
              </q-item-section>

              <q-item-section class="tw-font-medium">用户管理</q-item-section>
            </q-item>
            <q-item
              clickable
              :inset-level="0.2"
              :to="{
                name: 'node-admin',
                params: { orgId: store.getters.currentOrg },
              }"
            >
              <q-item-section avatar>
                <q-icon name="dynamic_form" />
              </q-item-section>

              <q-item-section class="tw-font-medium">节点管理</q-item-section>
            </q-item>
          </q-expansion-item>
          <q-separator class="tw-bg-stone-800" />

          <q-item clickable :to="{ name: 'logout' }">
            <q-item-section avatar>
              <q-icon name="logout" />
            </q-item-section>

            <q-item-section class="tw-font-medium">退出</q-item-section>
          </q-item>
        </q-list>
      </q-scroll-area>

      <UserBaseInfo
        class="tw-absolute tw-bottom-0 tw-left-0 tw-w-full"
        style="height: 80px"
      ></UserBaseInfo>
    </q-drawer>

    <q-page-container class="tw-w-full tw-h-full tw-box-border">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
  import { useQuasar } from 'quasar';
  import { computed, ref, watchEffect } from 'vue';
  import { useRoute } from 'vue-router';

  import OrgSelect from './OrgSelect.vue';
  import UserBaseInfo from './UserBaseInfo.vue';

  import { getServiceVersion } from '@/api/version';
  import { useStore } from '@/store/index';

  const $q = useQuasar();
  const drawer = ref(true);
  if ($q.screen.lt.md) {
    drawer.value = false;
  }
  const miniState = ref(false);
  const adminMenuExpanded = ref(false);
  const store = useStore();
  const route = useRoute();
  watchEffect(() => {
    if (!route.name) return;
    const adminMenuItemRouteNames = ['user-admin', 'node-admin'];
    adminMenuExpanded.value = adminMenuItemRouteNames.includes(
      route.name.toString(),
    );
  });

  const showMenuOrgSelector = computed(() => {
    if ($q.screen.lt.md) {
      return drawer.value;
    }
    return !miniState.value;
  });

  const version = ref('');
  const getVersion = async () => {
    const v = await getServiceVersion();
    version.value = v;
  };
  getVersion();

  const notificationSetting = computed(() => store.getters.notificationSetting);

  function switchNotificationSetting() {
    store.dispatch('changeNotificationStatus');
  }

  // const userOrgOptions = computed(() => {
  //   const tmp: { label: string; value: string }[] = [];
  //   store.state.Root.userOrgs?.forEach((v, k, a) => {
  //     tmp.push({
  //       label: v.title,
  //       value: v.id,
  //     });
  //   });
  //   return tmp;
  // });
</script>
