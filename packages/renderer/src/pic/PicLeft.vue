<script setup lang="ts">
import { computed, ref } from 'vue'

import 'ant-design-vue/es/tree/style/css'
import MySwitchTab from '../layout/MySwitchTab.vue'
import { useAppStore, useWinStore } from '../store'

const treeref = ref()
const winStore = useWinStore()
const appStore = useAppStore()
const TreeHeight = computed(() => winStore.height - 42 - 56 - 24 - 4)

const switchvalues = [
  { key: 'allpic', title: '全部照片', alt: '' },
  { key: 'xiangce', title: '我的相册', alt: '' }
]
</script>

<template>
  <div style="width: 100%; height: 100%; overflow: hidden; min-width: 300px">
    <div class="headswitch">
      <div class="bghr"></div>
      <div class="sw">
        <MySwitchTab :name="'picleft'" :tabs="switchvalues" :value="appStore.GetAppTabMenu" @update:value="(val:string)=>appStore.toggleTabMenu('pic', val)" />
      </div>
    </div>
    <div class="treeleft">
      <a-tabs type="text" :direction="'horizontal'" class="hidetabs" :justify="true" :active-key="appStore.GetAppTabMenu">
        <a-tab-pane key="allpic" title="1"><div style="text-align: center;">全部照片--还没做</div></a-tab-pane>
        <a-tab-pane key="xiangce" title="2"><div style="text-align: center;">我的相册--还没做</div></a-tab-pane>
      </a-tabs>
    </div>
  </div>
</template>

<style>
.treeleft {
  width: calc(100% + 22px);
}
</style>
