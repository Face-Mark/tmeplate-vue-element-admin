<!-- 上传文件(单文件) -->
<template>
  <div>
    <el-upload
      ref="uploadaaa"
      class="upload-demo"
      action="#"
      :http-request="httpRequestPic"
      :list-type="listType"
      :file-list="fileListPic"
      :on-change="handleChange"
      :on-remove="handleRemovePic"
      :before-upload="beforeAvatarUploadPic"
      :limit="limit"
      :auto-upload="true"
      :on-preview="Download"
    >
      <el-button slot="trigger" icon="el-icon-upload" size="small" type="primary">{{ btntxt }}</el-button>
      <!-- <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button> -->

    </el-upload>
    <div slot="tip" class="el-upload__tip">
      <p>{{ text1 }}</p>
      <p>{{ text2 }}</p>
    </div>
  </div>
</template>

<script>
import { putObject } from '@/utils/upload'

export default {
  name: 'UpLoadPicOnly',
  props: {
    isBig: {
      type: Boolean,
      default: false
    },
    maxSzie: {
      type: Number,
      default: 1
    },
    value: {
      type: Array,
      default: () => []
    },
    typePicList: {
      type: Array,
      default: () => ['image/jpeg', 'image/png', 'image/jpg']
    },
    text1: {
      type: String,
      default: '注：推荐尺寸40px*40px'
    },
    text2: {
      type: String,
      default: '仅支持jpeg，png，jpg格式，文件小于1M（方形图）'
    },
    btntxt: {
      type: String,
      default: '选择文件'
    },
    listType: {
      type: String,
      default: 'text'
    },
    limit: {
      type: Number,
      default: 100
    },
    // 保存的文件路径
    savePath: {
      type: String,
      default: '/wanma/pic'
    }
  },
  data() {
    return {
    }
  },
  computed: {
    fileListPic: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    }
  },
  methods: {
    // 文件上传
    submitUpload() {
      console.log('上传')
      this.$refs.uploadaaa.submit()
    },
    httpRequestPic(data) {
      console.log('data', data)
      const file = data.file
      const year = new Date().getFullYear(); const month = new Date().getMonth() + 1; const day = new Date().getDate(); const timeSing = new Date().getTime()
      const fileName = `${this.savePath}/${year}${month}/${day}/${timeSing}.${file.name.split('.').pop()}`
      // const fileName = `${this.savePath}/${year}${month}/${day}/${file.name}`
      console.log('fileName', fileName)
      putObject(file, fileName, res => {
        console.log('上传res', res)
        console.log('fileListPic', this.fileListPic)
        const obj = {}
        obj.name = `${timeSing}.${file.name.split('.').pop()}`
        obj.url = res
        this.fileListPic = [obj]
        this.$emit('httpRequestPic', res)
        // this.changeELupload('add')
      })
    },
    handleChange(file) {
      console.log('选择文件', file)
      const obj = {}
      obj.name = file.name
      obj.url = '#'
      this.fileListPic = [obj]
    },
    handleRemovePic(file) {
      console.log('removefile', file)
      const findex = this.fileListPic.map(f => f.uid).indexOf(file.uid)
      this.fileListPic.splice(findex, 1)
      console.log('delfileListPic', this.fileListPic)
      this.$emit('handleRemovePic')
      // this.changeELupload('cut')
    },
    beforeAvatarUploadPic(file) {
      console.log('file', file)
      const isJPG = this.typePicList.includes(file.type)
      const isLage = file.size > this.maxSzie * 1024 * 1024
      if (!isJPG) {
        this.$message.error('上传图片只支持 jpeg，png，jpg格式!')
        return false
      }
      if (isLage) {
        this.$message.error(`文件大于${this.maxSzie}M，请重新上传！`)
        return false
      }
    },
    // 控制elupload样式
    changeELupload(type) {
      switch (type) {
        case 'add':
          this.$refs.upload.$el.querySelector('.el-upload--picture-card').style.display = 'none'
          break
        case 'cut':
          setTimeout(() => {
            this.$refs.upload.$el.querySelector('.el-upload--picture-card').style.display = 'inline-block'
          }, 1000)
          break
        default:
          break
      }
    },
    // 下载
    Download(ev) {
      console.log('ev', ev)
      if (!ev.url) return
      window.open(ev.url)
    }
  }
}
</script>

<style lang="scss" scoped>
.el-upload__tip{color: #999;}
</style>

