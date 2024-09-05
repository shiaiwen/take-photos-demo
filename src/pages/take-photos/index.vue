<template>
  <view class="wrapper">
    <camera
      device-position="back"
      flash="off"
      style="width: 100%; height: 300px"
      @error="error"
    ></camera>
    <button type="primary" @click="takePhoto('single')" class="btn">
      拍单张
    </button>
    <button type="primary" @click="takePhoto('multiple')" class="btn">
      拍多张
    </button>
    <view class="" v-if="mode === 'single'">
      <image mode="widthFix" :src="src"></image>
    </view>
    <template v-if="mode === 'multiple'">
      <view v-for="(image, index) in images" :key="index">
        <image :src="image" class="photo" />
      </view>
    </template>
  </view>
</template>

<script>
export default {
  data() {
    return {
      src: '', // 单张图片路径
      mode: 'single', // 拍照模式  single 单张 'multiple'
      images: [] // 多张图片数组
    }
  },
  created() {},
  methods: {
    takePhoto(mode) {
      this.mode = mode
      const ctx = uni.createCameraContext()
      ctx.takePhoto({
        quality: 'high',
        success: res => {
          if (mode === 'single') {
            this.src = res.tempImagePath
          } else {
            this.images.push(res.tempImagePath)
          }
        }
      })
    },
    error(e) {
      console.log(e.detail)
    }
  }
}
</script>

<style scope lang="scss">
.btn {
  margin: 10px;
  padding: 0 10px;
  line-height: 30px;
  height: 30px;
  text-align: center;
  border-radius: 5px;
}
</style>
