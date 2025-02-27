<script lang="ts">
import { IAliFileAudioMeta, IAliFileItem, IAliFileVideoMeta } from '@/aliapi/alimodels'
import AliFile from '@/aliapi/file'
import { IVideoXBTUrl } from '@/aliapi/models'
import { useAppStore, useSettingStore } from '@/store'
import { copyToClipboard } from '@/utils/electronhelper'
import { humanSize, humanTime } from '@/utils/format'
import message from '@/utils/message'
import { computed, defineComponent, onMounted, ref } from 'vue'
const { getCurrentWindow } = window.require('@electron/remote')
import domtoimage from 'dom-to-image'

export default defineComponent({
  setup() {
    const handleHideClick = (_e: any) => {
      getCurrentWindow().close()
    }
    const appStore = useAppStore()
    const settingStore = useSettingStore()
    const loading = ref(false)
    const preview = ref(false)
    const error = ref('')
    const fileinfo = ref<IAliFileItem>()
    const rowNum = ref(4)
    const imagelist = ref<IVideoXBTUrl[]>([])

    onMounted(() => {
      handleWinSizeClick(settingStore.uiXBTWidth || 960)
      loadXBT()

      let name = '视频雪碧图 ' + (appStore.pageVideoXBT?.file_name || '')
      setTimeout(() => {
        document.title = name
      }, 1000)
      setTimeout(() => {
        document.title = name
      }, 10000)
    })

    const loadXBT = async () => {
      const pageVideoXBT = appStore.pageVideoXBT!

      
      loading.value = true
      let alifile = await AliFile.ApiFileInfo(pageVideoXBT.user_id, pageVideoXBT.drive_id, pageVideoXBT.file_id)

      if (alifile == undefined) {
        message.error('错误的文件')
        error.value = '错误的文件'
        return
      }
      const duration = Math.ceil(Number.parseFloat(alifile.video_media_metadata?.duration?.toString() || alifile.video_preview_metadata?.duration?.toString() || '0'))
      if (duration <= 0) {
        message.error('错误的文件(视频时长错误)')
        error.value = '错误的文件(视频时长错误)'
        fileinfo.value = alifile
        return
      }

      let x = Math.max(alifile?.video_media_metadata?.width || 0, alifile?.video_preview_metadata?.width || 0)
      let y = Math.max(alifile?.video_media_metadata?.height || 0, alifile?.video_preview_metadata?.height || 0)

      let uiXBTNumber = settingStore.uiXBTNumber || 36
      
      AliFile.ApiBiXueTuBatch(pageVideoXBT.user_id, pageVideoXBT.drive_id, pageVideoXBT.file_id, duration, uiXBTNumber, 720).then((data) => {
        fileinfo.value = alifile
        rowNum.value = x > y ? 3 : 4
        loading.value = false
        imagelist.value = data
      })
    }

    const handleWinSizeClick = (size: number) => {
      settingStore.uiXBTWidth = size
      let win = getCurrentWindow()
      if (win.isMaximized()) win.restore()
      let wsize = win.getSize()
      win.setSize(size + 32, wsize[1])
    }

    const handleCopyM3U8Click = () => {
      const pageVideoXBT = appStore.pageVideoXBT!
      AliFile.ApiVideoPreviewUrl(pageVideoXBT.user_id, pageVideoXBT.drive_id, pageVideoXBT.file_id).then((data) => {
        if (data && data.url) {
          copyToClipboard(data.url)
          message.success('视频的M3U8链接已复制，4小时内有效')
        } else {
          message.error('视频的M3U8链接获取失败，请稍后重试')
        }
      })
    }
    const handleCopyDownClick = () => {
      const pageVideoXBT = appStore.pageVideoXBT!
      AliFile.ApiFileDownloadUrl(pageVideoXBT.user_id, pageVideoXBT.drive_id, pageVideoXBT.file_id, 14400).then((data) => {
        if (data && typeof data !== 'string' && data.url) {
          copyToClipboard(data.url)
          message.success('视频的下载链接已复制，4小时内有效')
        } else {
          message.error('视频的下载链接获取失败，请稍后重试')
        }
      })
    }

    const handleSaveImageClick = () => {
      loading.value = true
      var node = document.getElementById('docxbt')
      var width = document.body.clientWidth
      domtoimage
        .toPng(node, { bgcolor: '#23232e' })
        .then((dataUrl: any) => {
          loading.value = false
          var link = document.createElement('a')
          link.download = appStore.pageVideoXBT?.file_name + '_' + (width - 32).toString() + '.png'
          link.href = dataUrl
          link.click()
        })
        .catch((error: any) => {
          loading.value = false
          message.error('生成雪碧图出错 ' + (error.message || ''))
        })
    }

    const getFenBianLv = computed(() => {
      let x = Math.max(fileinfo.value?.video_media_metadata?.width || 0, fileinfo.value?.video_preview_metadata?.width || 0)
      let y = Math.max(fileinfo.value?.video_media_metadata?.height || 0, fileinfo.value?.video_preview_metadata?.height || 0)
      return x.toString() + 'x' + y.toString()
    })

    const getVideoInfo = computed(() => {
      let info = ''
      let metas: IAliFileVideoMeta[] = []
      if (fileinfo.value?.video_media_metadata?.video_media_video_stream) {
        if (Array.isArray(fileinfo.value?.video_media_metadata?.video_media_video_stream)) {
          metas = fileinfo.value?.video_media_metadata?.video_media_video_stream
        } else {
          metas = [fileinfo.value?.video_media_metadata?.video_media_video_stream]
        }
      }

      for (let i = 0, maxi = metas.length; i < maxi; i++) {
        let meta = metas[i]
        if (meta.code_name == 'mjpeg') continue
        let one = ''

        if (meta.code_name) one += '    编码=' + meta.code_name.toUpperCase()
        if (meta.bitrate) {
          let bit = parseInt(meta.bitrate)
          let bitdw = 'bps'
          if (bit / 1000 > 40) {
            bit = bit / 1000
            bitdw = 'kbps'
          }
          if (bit / 1000 > 40) {
            bit = bit / 1000
            bitdw = 'mbps'
          }
          one += '    码率=' + Math.floor(bit) + bitdw
        }
        if (meta.fps) {
          one += '    帧率=' + meta.fps
          let fps = meta.fps.split('/')
          if (fps.length == 2) {
            let fpsn = parseInt(fps[0]) / parseInt(fps[1])
            one += ' (' + fpsn.toFixed(1) + ')'
          }
        }
        if (i > 0) info += ';'
        info += one
      }
      info = info.trimStart()

      if (!info) return ''
      return info
    })

    const getAudioInfo = computed(() => {
      let info = ''
      let metas: IAliFileAudioMeta[] = []
      if (fileinfo.value?.video_media_metadata?.video_media_audio_stream) {
        if (Array.isArray(fileinfo.value?.video_media_metadata?.video_media_audio_stream)) {
          metas = fileinfo.value?.video_media_metadata?.video_media_audio_stream
        } else {
          metas = [fileinfo.value?.video_media_metadata?.video_media_audio_stream]
        }
      }

      let audlist: string[] = []
      for (let i = 0, maxi = metas.length; i < maxi; i++) {
        let meta = metas[i]
        let one = ''
        if (meta.code_name) one += '    编码=' + meta.code_name.toUpperCase()

        if (meta.bit_rate) {
          let bit = parseInt(meta.bit_rate)
          let bitdw = 'bps'
          if (bit / 1000 > 8) {
            bit = bit / 1000
            bitdw = 'kbps'
          }
          if (bit / 1000 > 8) {
            bit = bit / 1000
            bitdw = 'mbps'
          }
          one += '    码率=' + Math.floor(bit) + bitdw
        }
        if (meta.sample_rate) one += '    采样率=' + meta.sample_rate + 'Hz'
        if (meta.channels) one += '    声道数=' + meta.channels
        if (meta.channel_layout) one += '    声道=' + meta.channel_layout.replace('stereo', '立体声').replace('mono', '单声道').replace('quad', '四通道')
        if (audlist.includes(one) == false) {
          audlist.push(one)
        }
      }
      if (metas.length > 1) info = '    音轨=' + metas.length.toString()
      info += audlist.join(';')
      info = info.trimStart()
      if (!info) return ''
      return info
    })

    const getPreviewInfo = computed(() => {
      let info = ''
      let meta = fileinfo.value?.video_preview_metadata
      if (meta) {
        let one = ''
        if (meta.video_format) one += '    视频编码=' + meta.video_format.toUpperCase()
        if (meta.bitrate) {
          let bit = parseInt(meta.bitrate)
          let bitdw = 'bps'
          if (bit / 1000 > 100) {
            bit = bit / 1000
            bitdw = 'kbps'
          }
          if (bit / 1000 > 100) {
            bit = bit / 1000
            bitdw = 'mbps'
          }
          one += '    总码率=' + Math.floor(bit) + bitdw
        }
        if (meta.frame_rate) {
          one += '    帧率=' + meta.frame_rate
          let fps = meta.frame_rate.split('/')
          if (fps.length == 2) {
            let fpsn = parseInt(fps[0]) / parseInt(fps[1])
            one += ' (' + fpsn.toFixed(1) + ')'
          }
        }
        if (meta.audio_format) one += '    音频编码=' + meta.audio_format.toUpperCase()
        info = one
      }
      info = info.trimStart()
      if (!info) return ''
      return info
    })

    return {
      handleHideClick,
      appStore,
      settingStore,
      loading,
      preview,
      rowNum,
      fileinfo,
      imagelist,
      handleWinSizeClick,
      handleCopyM3U8Click,
      handleCopyDownClick,
      handleSaveImageClick,
      humanSize,
      humanTime,
      getFenBianLv,
      getVideoInfo,
      getAudioInfo,
      getPreviewInfo
    }
  }
})
</script>

<template>
  <a-layout style="height: 100vh" draggable="false">
    <a-layout-header id="xbyhead" draggable="false">
      <div id="xbyhead2" class="q-electron-drag">
        <a-button type="text" tabindex="-1">
          <i class="iconfont iconfile_video"></i>
        </a-button>
        <div class="title">视频雪碧图 {{ appStore.pageVideoXBT?.file_name || '' }}</div>
        <div class="flexauto"></div>
        <a-button type="text" tabindex="-1" @click="handleHideClick">
          <i class="iconfont iconclose"></i>
        </a-button>
      </div>
    </a-layout-header>
    <a-layout-content style="height: calc(100vh - 42px); padding: 12px 6px 12px 16px">
      <div class="doc-preview" id="doc-preview" style="width: 100%; height: 100%; overflow: auto">
        <div className="xbtbtns settingbody">
          <a-radio-group type="button" size="small" tabindex="-1" :model-value="settingStore.uiXBTWidth" @update:model-value="handleWinSizeClick($event as number)">
            <a-radio tabindex="-1" :value="720">720P</a-radio>
            <a-radio tabindex="-1" :value="960">960P</a-radio>
            <a-radio tabindex="-1" :value="1080">1080P</a-radio>
            <a-radio tabindex="-1" :value="1280">1280P</a-radio>
          </a-radio-group>

          <div style="margin-right: 12px"></div>

          <a-button type="outline" size="small" tabindex="-1" :loading="loading" @click="handleSaveImageClick" title="下载雪碧图到本地"> <i class="iconfont icondownload"></i>保存雪碧图 </a-button>

          <div style="flex-grow: 1"></div>
          <a-button v-if="false" type="outline" size="small" tabindex="-1" @click="handleCopyM3U8Click" title="复制M3U8链接"> <i class="iconfont iconlink2"></i>M3U8链接 </a-button>

          <div style="margin-right: 12px"></div>
          <a-button v-if="false" type="outline" size="small" tabindex="-1" @click="handleCopyDownClick" title="复制原文件链接"> <i class="iconfont iconlink2"></i>下载链接 </a-button>

          <div style="margin-right: 12px"></div>
        </div>

        <div id="docxbt">
          <div className="xbtinfo" style="color: #ffffffd9; padding: 24px 8px 16px 16px; background: #17171f; border-radius: 4px; white-space: pre-wrap; margin-bottom: 2px">
            <h3 style="color: #fffffff2; font-size: 18px; font-weight: 500; line-height: 1.40625">{{ fileinfo?.name }}</h3>
            <div>
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 文件大小 </span>
              ：{{ humanSize(fileinfo?.size || 0) }}
            </div>
            <div>
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 分辨率 </span>
              ：{{ getFenBianLv }}
            </div>
            <div>
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 播放时长 </span>
              ：{{ humanTime(fileinfo?.video_media_metadata?.duration || fileinfo?.video_preview_metadata?.duration || 0) }}
            </div>

            <div v-if="getVideoInfo">
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 视频信息 </span>
              ：{{ getVideoInfo }}
            </div>

            <div v-if="getAudioInfo">
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 音频信息 </span>
              ：{{ getAudioInfo }}
            </div>

            <div v-if="getPreviewInfo">
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 预览信息 </span>
              ：{{ getPreviewInfo }}
            </div>
            <div v-if="fileinfo?.video_media_metadata?.time || ''">
              <span className="xinfo" style="display: inline-block; width: 66px; text-align: right"> 创建时间 </span>
              ：{{ fileinfo?.video_media_metadata?.time }}
            </div>
          </div>
          <div className="xbtvideo" style="display: flex; flex-wrap: wrap; border: 2px solid #17171f; border-bottom: 24px solid #17171f; padding: 16px; border-radius: 4px; background: #17171f">
            <a-image-preview-group @visible-change="(val:boolean)=>{preview=val}">
              <div v-for="(item, index) in imagelist" className="xbtimage" :key="'xbt-' + index.toString()" :style="{ width: rowNum == 4 ? '25%' : '33.333%' }">
                <a-image width="100%" :src="item.url" />
                <div className="xbttime" style="text-align: center; color: #ffffffa6">
                  {{ item.time }}
                </div>
              </div>
            </a-image-preview-group>
          </div>
        </div>
      </div>
    </a-layout-content>
  </a-layout>
</template>

<style>
.xbtbtns {
  margin: 14px 0 36px 0;
  display: flex;
}

.xbtimage {
  display: flex;
  flex-direction: column;
  padding-bottom: 8px;
  border: 2px solid #17171f;
  min-height: 100px;
}
</style>
