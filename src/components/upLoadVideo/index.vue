<!-- 图片上传-单图-样式40*40-无压缩 -->
<template>
  <div class="upload-video">
    <el-upload
      ref="upload"
      action="#"
      :http-request="httpRequestVideo"
      list-type="picture-card"
      :show-file-list="false"
      :file-list="fileListVideo"
      :before-upload="beforeAvatarUploadVideo"
      :limit="1"
    >
      <i v-if="!videoUrl" class="el-icon-plus" />
      <div v-if="videoUrl&&fileType==='video'" class="up_v_play">
        <video class="up_v_play_01" :src="videoUrl">您的浏览器不支持视频播放</video>
        <div class="up_v_play_02"><i class="el-icon-refresh-right" />重新上传</div>
        <div class="up_v_play_03" @click.stop="playVideo"><i class="el-icon-video-play" />播放</div>
      </div>
      <div v-if="videoUrl&&fileType==='audio'" class="up_v_play_audio">
        <audio class="play_audio" :src="videoUrl" controls>您的浏览器不支持语音播放</audio>
        <div class="play_audio_02"><i class="el-icon-refresh-right" style="font-size:16px;" /><span>重新上传</span></div>
        <!-- <div class="up_v_play_03" @click.stop="playVideo"><i class="el-icon-video-play" />播放</div> -->
      </div>
    </el-upload>
    <div class="up_v_toast">
      <p />
      <p>{{ text1 }}</p>
    </div>
    <div v-if="videoLoading" class="up_v_progress">
      <el-progress class="up_v_progress_cir" type="circle" :percentage="progressVal" :width="100" />
    </div>
    <video-prop ref="videoPlay" :video-url="videoUrl" />
  </div>
</template>

<script>
import { putObject } from '@/utils/upload'
import { Accurac } from '@/utils'
import videoProp from './videoProp'

export default {
  name: 'UploadVideo',
  components: {
    videoProp
  },
  props: {
    text1: {
      type: String,
      default: '（视频大小不可大于300M）'
    },
    maxSzie: {
      type: Number,
      default: 300
    },
    value: {
      type: String,
      default: ''
    },
    typeVideoList: {
      type: Array,
      default: () => ['video/mp4']
    },
    fileType: {
      type: String,
      default: 'video'
    }
  },
  data() {
    return {
      progressVal: 0,
      videoLoading: false,
      tempVideoInfo: {},
      fileListVideo: []
    }
  },
  computed: {
    videoUrl: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    }
  },
  methods: {
    // 播放视频
    playVideo() {
      this.$refs.videoPlay.isShow = true
    },
    httpRequestVideo(data) {
      const file = data.file
      const year = new Date().getFullYear()
      const month = new Date().getMonth() + 1
      const day = new Date().getDate()
      const timeSing = new Date().getTime()
      const fileName = this.fileType === 'video'
        ? `/video/file/${year}${month}/${day}/${timeSing}.${file.name.split('.').pop()}`
        : `/audio/file/${year}${month}/${day}/${timeSing}.${file.name.split('.').pop()}`
      this.videoLoading = true
      putObject(file, fileName, res => {
        this.$refs.upload.clearFiles()
        const tempData = { ...this.tempVideoInfo, videoUrl: res }
        this.$emit('getVideoInfo', tempData)
        this.videoUrl = res
        this.videoLoading = false
      }, (val) => {
        this.progressVal = Accurac('Lower', (val * 100), 0, 0)
      })
    },
    beforeAvatarUploadVideo(file) {
      const isLage = file.size > 1024 * this.maxSzie * 1024
      const isVideo = this.typeVideoList.includes(file.type)
      const type = file.name.split('.').pop()
      if (!isVideo && this.fileType === 'video') {
        this.$message.error('上传视频仅支持 mp4 格式!')
        return false
      }
      if (this.fileType === 'audio' && type !== 'mp3') {
        this.$message.error('上传音频仅支持 mp3 格式!')
        return false
      }
      if (isLage && this.fileType === 'video') {
        this.$message.error(`上传的视频最大为${this.maxSzie}M!`)
        return false
      }
      if (isLage && this.fileType === 'audio') {
        this.$message.error(`上传的音频最大为${this.maxSzie}M!`)
        return false
      }
      const videoUrl = URL.createObjectURL(file)
      const videoObj = this.fileType === 'video' ? document.createElement('video') : document.createElement('audio')
      videoObj.onloadedmetadata = (evt) => {
        console.log(evt)
        URL.revokeObjectURL(videoUrl)
        this.tempVideoInfo = {
          fileWidth: Math.round(+videoObj.videoWidth),
          fileHeight: Math.round(+videoObj.videoHeight),
          fileSize: Math.round(+file.size),
          fileTime: Math.floor(+videoObj.duration)
        }
      }
      videoObj.src = videoUrl
      console.log(videoObj)
      videoObj.load()
    }
  }
}
</script>

<style lang="scss" scoped>
  .upload-video {
    .up_v_play_audio{
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      .play_audio{
        position: absolute;
        width: 280px;
        transform: scale(.7);
        transform-origin: left;
        top: 10px;
        left:2px;
      }
      .play_audio_02{
        width: 200px;
        height:30px;
        position: absolute;
        top:30px;
        display: inline;
        color: rgb(49, 129, 235);
      }
    }
    .up_v_play {
      background-color: #000;
      position: absolute;
      width: 200px;
      height: 113px;
      z-index: 2;
      left: 0;
      top: 0;
      border-radius: 5px;
      overflow: hidden;
      .up_v_play_01 {
        width: 200px;
        height: 113px;
      }
      .up_v_play_02 {
        font-size: 13px;
        display: flex;
        align-items: center;
        color: #fff;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        i {
          font-size: 18px;
          margin-right: 5px;
          color: #fff;
        }
      }
      .up_v_play_03 {
        font-size: 13px;
        height: 30px;
        position: absolute;
        top: 8px;
        right: 8px;
        display: flex;
        color: #fff;
        align-items: center;
        i {
          font-size: 18px;
          margin-right: 5px;
          color: #fff;
        }
      }
    }
    .up_v_toast {
      padding-right: 50px;
      font-size: 12px;
      color: #999;
      position: absolute;
      background: #fff;
      z-index: 2;
      left: 206px;
      top: 0;
      p {
        padding-left: 17px;
        text-align: left;
        display: block;
        height: 40px;
        &:nth-child(1) {
          margin-bottom: 36px;
        }
      }
    }
    .up_v_progress {
      z-index: 99;
      position: absolute;
      width: 200px;
      height: 113px;
      left: 0;
      top: 0;
      background-color: rgba(255, 255, 255, 1);
      border-radius: 6px;
      border: 1px solid rgba(0, 0, 0, 0.1);
      transform: scale(1.02);
      .up_v_progress_cir {
        position: absolute;
        left: 50px;
        top: 3px;
      }
    }
    ::v-deep {
      .el-loading-parent--relative {
        width: 200px;
      }
      .el-upload--picture-card {
        width: 200px;
        height: 113px;
        background: rgba(249, 249, 249, 1);
        line-height: 113px;
      }
      .el-upload{
        width: 200px;
        height: 113px;
        line-height: 109px;
      }
      .el-upload-list--picture-card .el-upload-list__item{
        width: 200px;
        height: 113px;
        line-height: 100px;
      }
      .el-upload-list--picture-card .el-upload-list__item-thumbnail{
        width: 200px;
        height: 113px;
        line-height: 100px;
      }
      .avatar{
        width: 200px;
        height: 113px;
      }
      .el-icon-upload-success {
        position: absolute;
        top: 0px;
        right: 12px;
      }
    }
  }
</style>

