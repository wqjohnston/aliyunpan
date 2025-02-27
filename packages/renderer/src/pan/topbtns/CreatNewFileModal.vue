<script lang="ts">
import AliUploadMem from '@/aliapi/uploadmem'
import { usePanTreeStore } from '@/store'
import message from '@/utils/message'
import { modalCloseAll } from '@/utils/modal'
import { CheckFileName, ClearFileName } from '@/utils/utils'
import { computed, defineComponent, ref, reactive } from 'vue'
import PanDAL from '../pandal'

export default defineComponent({
  props: {
    visible: {
      type: Boolean,
      required: true
    }
  },
  setup(props) {
    const okLoading = ref(false)
    const formRef = ref()
    const form = reactive({
      filename: '',
      filecontext: ''
    })

    const handleClose = () => {
      
      if (okLoading.value) okLoading.value = false
      formRef.value.resetFields()
    }

    const rules = [
      { required: true, message: '文件名必填' },
      { minLength: 1, message: '文件名不能为空' },
      { maxLength: 100, message: '文件名太长(100)' },
      {
        validator: (value: string, cb: any) => {
          let chk = CheckFileName(value)
          if (chk) cb('文件名' + chk)
        }
      }
    ]

    return { okLoading, form, formRef, handleClose, rules }
  },
  methods: {
    handleHide() {
      modalCloseAll()
    },
    handleOK() {
      this.formRef.validate((data: any) => {
        if (data) return 

        const pantreeStore = usePanTreeStore()
        if (!pantreeStore.user_id || !pantreeStore.drive_id || !pantreeStore.selectDir.file_id) {
          message.error('新建文件失败 父文件夹错误')
          return
        }

        let newname = ClearFileName(this.form.filename)
        if (!newname) {
          message.error('新建文件失败 文件名不能为空')
          return
        }

        this.okLoading = true
        AliUploadMem.UploadMem(pantreeStore.user_id, pantreeStore.drive_id, pantreeStore.selectDir.file_id, newname, this.form.filecontext)
          .then((data) => {
            this.okLoading = false
            if (data && data == 'success') {
              
              PanDAL.aShowDir('', 'refresh', false)
              message.success('新建文件 成功')
              modalCloseAll()
            } else {
              message.error('新建文件 失败 ' + data)
            }
          })
          .catch((e: any) => {
            this.okLoading = false
            message.error('新建文件 失败 ' + (e.message || ''))
          })
      })
    }
  }
})
</script>

<template>
  <a-modal :visible="visible" modal-class="modalclass" @cancel="handleHide" @close="handleClose" :footer="false" :unmount-on-close="true" :mask-closable="false">
    <template #title>
      <span class="modaltitle">新建文本文件</span>
    </template>
    <div class="modalbody" style="width: 80vw; max-width: 860px; height: calc(80vh - 100px)">
      <a-form ref="formRef" :model="form" layout="vertical" style="height: 100%; display: flex">
        <a-form-item field="filename" :rules="rules">
          <template #label>文件名：<span class="opblue" style="margin-left: 16px; font-size: 12px"> 不要有特殊字符 &lt; > : * ? \\ / \' " </span> </template>
          <a-input v-model.trim="form.filename" placeholder="例如：今天工作计划.txt" allow-clear autofocus="autofocus" />
        </a-form-item>
        <a-form-item field="filecontext" label="文件内容：" class="textareafill">
          <a-textarea v-model="form.filecontext" placeholder="粘贴内容" show-word-limit @keydown="(e) => e.stopPropagation()" />
        </a-form-item>
      </a-form>
    </div>
    <div class="modalfoot">
      <div style="flex-grow: 1"></div>
      <a-button v-if="!okLoading" type="outline" size="small" @click="handleHide">取消</a-button>
      <a-button type="primary" size="small" :loading="okLoading" @click="handleOK">创建</a-button>
    </div>
  </a-modal>
</template>

<style>
.textareafill {
  flex-grow: 1;
  display: flex !important;
  flex-direction: column;
}
.textareafill .arco-form-item-wrapper-col {
  flex-grow: 1;
}
.textareafill .arco-form-item-wrapper-col .arco-form-item-content-wrapper,
.textareafill .arco-form-item-wrapper-col .arco-form-item-content,
.textareafill .arco-form-item-wrapper-col .arco-textarea-wrapper,
.textareafill .arco-form-item-wrapper-col .arco-textarea {
  height: 100%;
}
</style>
