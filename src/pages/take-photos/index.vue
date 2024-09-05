<template>
  <view class="wrapper">
    <camera
      :device-position="devicePosition"
      flash="off"
      style="width: 100%; height: 300px"
      @error="error"
    ></camera>
    <button type="primary" class="btn" @click="takePhoto('single')">
      拍单张
    </button>
    <button type="primary" class="btn" @click="takePhoto('multiple')">
      拍多张
    </button>
    <button type="primary" class="btn" @click="changeDevicePosition">
      反转摄像头
    </button>
    <button type="primary" class="btn" @click="reset">reset</button>
    <button type="primary" class="btn" @click="upload">upload</button>

    <view v-if="mode === 'single'" class="">
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
      images: [], // 多张图片数组
      devicePosition: 'back'
    }
  },
  created() {},
  methods: {
    // https://zh.uniapp.dcloud.io/component/camera.html#camera
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

    reset() {
      this.src = ''
      this.images = []
    },
    changeDevicePosition() {
      console.log('点击事件')
      const { devicePosition: dp } = this
      this.devicePosition = dp === 'back' ? 'front' : 'back'
    },
    error(e) {
      console.log(e.detail)
    },

    // 伪代码  上传图片到服务端
    upload() {
      // 2. 上传图片
      const { src, images } = this
      if (this.mode === 'single') {
        uploadImage(src)
      } else {
        uploadFiles(images)
      }
    },

    //  上传单图片函数
    uploadImage(filePath) {
      // 将图片转换为可以上传的格式，例如使用FormData
      const formData = new FormData()
      formData.append('file', filePath)
      // 可以添加其他表单数据
      formData.append('user', 'testUser')

      // 发起网络请求上传图片
      uni.uploadFile({
        url: 'https://example.com/upload', // 服务器上传图片的API地址
        filePath,
        name: 'file', // 必须与后端约定的字段名一致
        formData, // 需要上传的额外表单数据
        success(uploadRes) {
          // 服务器返回的结果
          console.log(uploadRes.data)
        },
        fail(uploadErr) {
          // 上传失败
          console.error(uploadErr)
        }
      })
    },
    // 多文件上传
    uploadFiles(filePaths) {
      const uploadTasks = filePaths.map(
        (filePath, index) =>
          new Promise((resolve, reject) => {
            uni.uploadFile({
              url: 'https://example.com/upload', // 服务器上传接口地址
              filePath,
              name: `file${index}`, // 文件对应的 key , 开发者在服务器端通过这个 key 可以获取到文件二进制内容
              formData: {
                user: 'test' // 需要提交的额外表单数据
              },
              success: uploadRes => {
                console.log('文件上传成功:', uploadRes)
                resolve(uploadRes)
              },
              fail: err => {
                console.error('文件上传失败:', err)
                reject(err)
              }
            })
          })
      )

      Promise.all(uploadTasks)
        .then(results => {
          console.log('所有文件上传成功:', results)
        })
        .catch(err => {
          console.error('文件上传出错:', err)
        })
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
