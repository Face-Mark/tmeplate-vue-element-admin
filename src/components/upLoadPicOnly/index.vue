<!-- 图片上传-单图-样式40*40-无压缩 -->
<template>
  <div class="upload-pic" :class="isBig ? 'form_data_upload_big' : 'form_data_upload_small'">
    <el-upload
      ref="upload"
      action="#"
      :http-request="httpRequestPic"
      :list-type="listType"
      :show-file-list="showFileList"
      :file-list="fileListPic"
      :on-remove="handleRemovePic"
      :before-upload="beforeAvatarUploadPic"
      :limit="1"
      class="editor-slide-upload"
    >
      <slot>
        <i class="el-icon-plus" />
      </slot>
    </el-upload>
    <div class="upload-toast">
      <p>{{ text1 }}</p>
      <p>{{ text2Formate }}</p>
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
      default: 0.3
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
      default: ``
    },
    showFileList: {
      type: Boolean,
      default: true
    },
    listType: {
      type: String,
      default: 'picture-card'
    }
  },
  data() {
    return {}
  },
  computed: {
    fileListPic: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    },
    text2Formate() {
      return this.text2 || `仅支持jpeg，png，jpg格式，文件小于${this.formatePicSize(this.maxSzie)}（方形图）`
    }
  },
  methods: {
    httpRequestPic(data) {
      const file = data.file
      const year = new Date().getFullYear()
      const month = new Date().getMonth() + 1
      const day = new Date().getDate()
      const timeSing = new Date().getTime()
      const fileName = `/wanma/pic/${year}${month}/${day}/${timeSing}.${file.name.split('.').pop()}`
      putObject(file, fileName, (res) => {
        const obj = {}
        obj.uid = file.uid
        obj.url = res
        this.fileListPic.push(obj)
        this.$emit('httpRequestPic', res)
        this.changeELupload('add')
      })
    },
    handleRemovePic(file) {
      const findex = this.fileListPic.map((f) => f.uid).indexOf(file.uid)
      this.fileListPic.splice(findex, 1)
      this.$emit('handleRemovePic')
      this.changeELupload('cut')
    },
    beforeAvatarUploadPic(file) {
      const isJPG = this.typePicList.includes(file.type)
      const isLage = file.size > this.maxSzie * 1024 * 1024
      if (!isJPG) {
        this.$message.error('上传图片只支持 jpeg，png，jpg格式!')
        return false
      }
      if (isLage) {
        this.$message.error(`文件大于${this.formatePicSize(this.maxSzie)}，请重新上传！`)
        return false
      }
    },
    formatePicSize(val) {
      return val > 1 ? `${val}M` : `${val * 1000}KB`
    },
    // 控制elupload样式
    changeELupload(type) {
      if (this.listType == 'picture-card') {
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
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.upload-pic {
  .upload-toast {
    min-width: 200px;
  }
}
</style>

