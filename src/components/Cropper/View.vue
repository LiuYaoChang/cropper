<template>
  <div
    class="cropper-view" 
    @click="handleContentClick">
    <move-container
      :width="width"
      :height="finalHeight"
      :moveAble="moveAble"
      @change="handleChange"
      @init="handleCropInit"
      :border="border"
      @click.stop>
      <img :src="src" alt="" class="img">
    </move-container>
  </div>
</template>
<script>
import MoveContainer from './Move';
export default {
  name: 'CropperView',
  props: {
    moveAble: {
      type: Boolean,
      default: false
    },
    border: {
      type: Boolean,
      default: false
    },
    width: {
      type: Number,
      default: 300
    },
    height: {
      type: Number,
      default: 300
    },
    autoResize: {
      type: Boolean,
      default: false
    },
    src: {
      type: String,
      required: true
    }
  },
  components: {
    MoveContainer
  },
  computed: {
    finalHeight() {
      if (this.autoResize && this.innerHeight > 0) {
        return this.innerHeight
      }
      return this.height;
    }
  },
  data () {
    return {
      focus: false,
      innerHeight: 0
    }
  },
  created() {
    this.getSize();
  },
  methods: {
    // 获取焦点
    handleContentClick() {
      this.focus = true;
    },
    getSize() {
      let image = new Image()
      image.src = this.src;
      image.crossOrigin = "anonymous";
      const vm = this;
      image.onload = function() {
        const ratio = this.naturalHeight / this.naturalWidth;
        vm.innerHeight = vm.width * ratio;
        // 图片携带的尺寸信息
        console.log('natural-size:::', this.naturalHeight, this.naturalWidth, ratio, vm.width, vm.innerHeight);
      }
    },
    // 图形旋转
    handleChange(data) {
      console.log('change:::', data);
      this.$emit('change', data);
    },
    handleCropInit(data) {
      this.$emit('init', data);
    },
  }
}
</script>
<style lang="scss">
.cropper-view {
  position: relative;
  & .img {
    width: 100%;
  }
}
</style>

