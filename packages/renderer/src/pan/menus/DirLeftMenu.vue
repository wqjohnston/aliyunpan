<script lang="ts">
import { defineComponent, ref } from 'vue'
import { menuTrashSelectFile, menuCopySelectedFile, menuCreatShare } from '../topbtns/topbtn'
import { modalRename, modalShuXing } from '../../utils/modal'
import PanDAL from '../pandal'
export default defineComponent({
  setup() {
    const handleRefresh = () => PanDAL.aShowDir('', 'refresh', false)
    const handleExpandAll = (isExpand: boolean) => PanDAL.ExpandDirAll(isExpand)
    const istree = true
    return { istree, handleRefresh, handleExpandAll, menuCreatShare, menuTrashSelectFile, menuCopySelectedFile, modalRename, modalShuXing }
  }
})
</script>

<template>
  <a-dropdown id="leftpanmenu" class="rightmenu" :popup-visible="true" tabindex="-1" :draggable="false" style="z-index: -1; left: -200px; opacity: 0">
    <template #content>
      <a-dsubmenu id="leftpansubzhankai" class="rightmenu" trigger="hover" tabindex="-1" :draggable="false">
        <template #default>
          <div @click.stop="() => {}">
            <span class="arco-dropdown-option-icon"><i class="iconfont iconfenzhi1"></i></span>Tree
          </div>
        </template>
        <template #content>
          <a-doption @click="handleRefresh">
            <template #icon> <i class="iconfont iconreload-1-icon" /> </template>
            <template #default>刷新</template>
          </a-doption>
          <a-doption @click="() => handleExpandAll(true)">
            <template #icon> <i class="iconfont iconArrow-Down2" /> </template>
            <template #default>展开全部</template>
          </a-doption>
          <a-doption @click="() => handleExpandAll(false)">
            <template #icon> <i class="iconfont iconArrow-Right2" /> </template>
            <template #default>折叠全部</template>
          </a-doption>
        </template>
      </a-dsubmenu>
      <a-doption>
        <template #icon> <i class="iconfont icondownload" /> </template>
        <template #default>下载</template>
      </a-doption>
      <a-doption @click="() => menuCreatShare(istree, 'pan')">
        <template #icon> <i class="iconfont iconfenxiang" /> </template>
        <template #default>分享</template>
      </a-doption>

      <a-dsubmenu id="leftpansubmove" class="rightmenu" trigger="hover" tabindex="-1" :draggable="false">
        <template #default>
          <div @click.stop="() => {}">
            <span class="arco-dropdown-option-icon"><i class="iconfont iconmoveto" style="opacity: 0.8"></i></span>移动
          </div>
        </template>
        <template #content>
          <a-doption @click="() => menuCopySelectedFile(istree, 'move')">
            <template #icon> <i class="iconfont iconscissor" /> </template>
            <template #default>移动到...</template>
          </a-doption>
          <a-doption @click="() => menuCopySelectedFile(istree, 'copy')">
            <template #icon> <i class="iconfont iconcopy" /> </template>
            <template #default>复制到...</template>
          </a-doption>
          <a-doption @click="() => menuTrashSelectFile(istree, false)" class="danger">
            <template #icon> <i class="iconfont icondelete" /> </template>
            <template #default>回收站</template>
          </a-doption>
          <a-doption @click="() => menuTrashSelectFile(istree, true)" class="danger">
            <template #icon> <i class="iconfont iconrest" /> </template>
            <template #default>彻底删除</template>
          </a-doption>
        </template>
      </a-dsubmenu>

      <a-doption @click="() => modalRename(istree, false)">
        <template #icon> <i class="iconfont iconedit-square" /> </template>
        <template #default>重命名</template>
      </a-doption>

      <a-doption @click="() => modalShuXing(istree, false)">
        <template #icon> <i class="iconfont iconshuxing" /> </template>
        <template #default>属性</template>
      </a-doption>
    </template>
  </a-dropdown>
</template>
<style></style>
