<script lang="ts">
import AliShare from '@/aliapi/share'
import { getFromClipboard } from '@/utils/electronhelper'
import message from '@/utils/message'
import { modalCloseAll, modalDaoRuShareLinkMulti } from '@/utils/modal'
import { defineComponent, ref, reactive } from 'vue'

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
      sharelink: '',
      password: ''
    })
    
    const FixFormate = (text: string, enmpty: boolean) => {
      let linktxt = ''
      let linkpwd = ''
      if (text && text.indexOf('密码') >= 0) text = text.replaceAll('密码', '提取码')
      if (text && text.indexOf('提取码') >= 0) {
        text = text.replace('提取码:', '提取码').replace('提取码：', '提取码').replace('提取码 ', '提取码').trim() 
        linkpwd = text.substr(text.indexOf('提取码') + '提取码'.length, 4)
      }

      if (text && text.length == 11) {
        linktxt = 'aliyundrive.com/s/' + text
      }

      if (text && text.indexOf('aliyundrive.com/s/') >= 0) {
        linktxt = 'aliyundrive.com/s/' + text.substr(text.indexOf('aliyundrive.com/s/') + 'aliyundrive.com/s/'.length, 11)
      }

      if (linktxt == '' && enmpty == false) linktxt = text
      return { linktxt, linkpwd }
    }
    
    const onPaste = (e: any) => {
      e.stopPropagation() 
      e.preventDefault() 
      let text = getFromClipboard()
      let link = FixFormate(text, true)
      form.sharelink = link.linktxt
      form.password = link.linkpwd
    }
    const handleOpen = () => {
      let text = getFromClipboard()
      let link = FixFormate(text, true)
      form.sharelink = link.linktxt
      form.password = link.linkpwd
    }

    const handleClose = () => {
      
      if (okLoading.value) okLoading.value = false
      formRef.value.resetFields()
    }

    return { okLoading, form, formRef, handleOpen, handleClose, onPaste }
  },
  methods: {
    handleHide() {
      modalCloseAll()
    },
    handleOK() {
      this.formRef.validate((data: any) => {
        if (data) return 

        if (this.form.sharelink.indexOf('aliyundrive.com/s/') < 0) {
          message.error('解析链接出错，必须为 aliyundrive.com/s/xxxxxxxxxxx 格式的链接')
          return
        }

        this.okLoading = true
        const sid = this.form.sharelink.split(/\.com\/s\/([\w]+)/)[1]
        AliShare.ApiGetShareToken(sid, this.form.password)
          .then((sharetoken) => {
            this.okLoading = false
            if (sharetoken == '' || sharetoken.startsWith('，')) {
              message.error('解析链接出错' + sharetoken)
            } else {                            
              modalCloseAll()
            }
          })
          .catch((e: any) => {
            this.okLoading = false
            message.error(e.message)
          })
      })
    },
    handleMulti() {
      modalDaoRuShareLinkMulti()
    }
  }
})
</script>

<template>
  <a-modal :visible="visible" modal-class="modalclass" @cancel="handleHide" @before-open="handleOpen" @close="handleClose" :footer="false" :unmount-on-close="true" :mask-closable="false">
    <template #title>
      <span class="modaltitle">批量导入阿里云盘分享链接</span>
    </template>
    <div class="modalbody" style="width: 440px">
      <a-form ref="formRef" :model="form" layout="horizontal" auto-label-width>
        <a-form-item
          field="sharelink"
          label="分享链接："
          :rules="[
            { required: true, message: '分享链接必填' },
            { minLength: 29, message: '分享链接太短' },
            { maxLength: 300, message: '分享链接太长(300)' },
            { match: /aliyundrive.com\/s\//, message: '必须是阿里云盘(aliyundrive.com/s/...)' },
            { match: /aliyundrive.com\/s\/[0-9a-zA-Z_]{11,}/, message: '格式错误：aliyundrive.com/s/umaDDMR7w4F' }
          ]"
        >
          <a-input v-model.trim="form.sharelink" @paste.stop.prevent="onPaste" placeholder="例如：aliyundrive.com/s/umaDDMR7w4F" allow-clear />
        </a-form-item>
        <a-form-item field="password" label="提取码：" :rules="[{ length: 4, message: '提取码必须是4个字符' }]">
          <a-input v-model.trim="form.password" placeholder="没有不填" allow-clear style="max-width: 100px" />
        </a-form-item>
      </a-form>
      <br />
    </div>
    <div class="modalfoot">
      <a-button type="outline" size="small" @click="handleMulti">批量导入</a-button>
      <div style="flex-grow: 1"></div>
      <a-button v-if="!okLoading" type="outline" size="small" @click="handleHide">取消</a-button>
      <a-button type="primary" size="small" :loading="okLoading" @click="handleOK">导入</a-button>
    </div>
  </a-modal>
</template>

<style></style>
