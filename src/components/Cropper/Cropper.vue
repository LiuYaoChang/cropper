<template>
  <div class="cropper-container">
    <div class="cropper-body">
      <div class="cropper-view__wrapper">
        <cropper-content-view :src="src" @change="handleImageRotate" @init="handleViewerInit"></cropper-content-view>
      </div>
      <div class="cropper-action">
        <cropper-container :border-color="cropBorderColor" @change="handleCropperChange" @init="handleCropInit" :left="200" :top="200"></cropper-container>
      </div>
    </div>
  </div>
</template>
<script>
import CropperContentView from './View';
import CropperContainer from './Move';
export default {
  name: 'Cropper',
  components: {
    CropperContainer,
    CropperContentView
  },
  props: {
    cropWidth: {
      type: Number,
      default: 100
    },
    cropHeight: {
      type: Number,
      default: 100
    },
    cropBorderColor: {
      type: String,
      default: '#333333'
    },
    contentWidth: {
      type: Number,
      default: 100
    },
    contentHeight: {
      type: Number,
      default: 100
    },
    src: {
      type: String,
      required: true
    }
  },
  data () {
    return {}
  },
  methods: {
    // 初始化
    handleViewerInit(data) {
      this.$emit('image-register', data);
    },
    // 初始化
    handleCropInit(data) {
      this.$emit('crop-register', data);
    },
    // 旋转图片
    handleImageRotate(data) {
      // console.log('image rotate:::', data)
      this.$emit('image-change', data);
    },
    // 裁剪框调整
    handleCropperChange(data) {
      console.log('cropper box:::', data);
      this.$emit('crop-change', data);
    }
  }
}
</script>
<style lang="scss">

</style>

