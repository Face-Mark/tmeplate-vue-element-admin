<!-- 图片上传-多张图-样式40*40-无压缩 -->
<template>
  <div class="upload-pic-more form_data_upload_small">
    <el-upload
      ref="upload"
      action="#"
      multiple
      :http-request="httpRequestPic"
      list-type="picture-card"
      :show-file-list="true"
      :file-list="fileListPic"
      :on-remove="handleRemovePic"
      :before-upload="beforeAvatarUploadPic"
      :limit="limit"
      class="editor-slide-upload"
    >
      <i class="el-icon-plus" />
    </el-upload>
  </div>
</template>

<script>
import { putObject } from '@/utils/upload'

export default {
  name: 'UpLoadPicMore',
  props: {
    limit: {
      type: Number,
      default: 1
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
    httpRequestPic(data) {
      const file = data.file
      const year = new Date().getFullYear(); const month = new Date().getMonth() + 1; const day = new Date().getDate(); const timeSing = new Date().getTime()
      const fileName = `/wanma/pic/${year}${month}/${day}/${timeSing}.${file.name.split('.').pop()}`
      putObject(file, fileName, res => {
        const obj = {}
        obj.uid = file.uid
        obj.url = res
        this.fileListPic.push(obj)
        this.$emit('httpRequestPic', res)
        this.changeELupload('add')
      })
    },
    handleRemovePic(file) {
      const findex = this.fileListPic.map(f => f.uid).indexOf(file.uid)
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
        this.$message.error(`文件大于${this.maxSzie}M，请重新上传！`)
        return false
      }
    },
    // 控制elupload样式
    changeELupload(type, el) {
      switch (type) {
        case 'add':
          if (this.fileListPic.length >= this.limit) {
            this.$refs.upload.$el.querySelector('.el-upload--picture-card').style.display = 'none'
          }
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
</script>

<style lang="scss" scoped>
  .upload-pic-more {
    .upload-toast {
      min-width: 200px;
    }
  }
</style>

