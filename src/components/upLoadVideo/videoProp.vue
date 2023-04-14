<template>
  <el-dialog
    :visible.sync="isShow"
    custom-class="video_prop"
    append-to-body
    :modal="false"
    :destroy-on-close="true"
    @opened="getVideoInfo"
  >
    <div class="video_prop_content">
      <em class="head-close" @click="isShow = false"><i class="el-icon-close" /></em>
      <video id="video" ref="video" controls :src="videoUrl" />
    </div>
  </el-dialog>
</template>

<script>

export default {
  name: 'VideoProp',
  // eslint-disable-next-line vue/require-prop-types
  props: ['videoUrl'],
  data() {
    return {
      isShow: false,
      detail: ''
    }
  },
  mounted() {
  },
  methods: {
    // 获取video宽高
    getVideoInfo() {
      const w = this.$refs.video.offsetWidth; const h = this.$refs.video.offsetHeight
      if (w / h >= 1) {
        this.$refs.video.width = 1000
      } else {
        this.$refs.video.height = 565
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  ::v-deep .video_prop {
    margin: 0 !important;
    position: absolute;
    width: 1000px;
    height: 565px;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: #000;
    overflow: hidden;
    .el-dialog__header {
      display: none !important;
    }
    .el-dialog__body {
      width: 100%;
      padding: 0;
    }
    .video_prop_content {
      overflow: hidden;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .head-close {
      position: absolute;
      right: 20px;
      top: 20px;
      cursor: pointer;
      display: block;
      width: 27px;
      height: 27px;
      z-index: 3;
      i {
        font-size: 22px;
        color: #fff;
      }
    }
  }
</style>
