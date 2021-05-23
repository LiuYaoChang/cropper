<template>
  <div class="vue-cropper__wrap">
    <div class="vue-cropper-container" ref="drawBox" :style="containerStyle">
      <cropper ref="vueCrop"
        :src="src"
        @image-change="handleImageRotate"
        @crop-change="handleCropChange"
        @crop-register="handleCropInit"
        @image-register="handleImageInit" />
    </div>
    <div class="vue-cropper-preview__wrap">
      <div class="final-image">
        <canvas ref="clip" id="drawer" width=""></canvas>
      </div>
      <div class="buttons">
        <slot name="button"></slot>
        <!-- <el-button type="primary" size="small">上传文件</el-button> -->
        <el-button type="primary" size="small" @click="handleDownload">下载文件</el-button>
      </div>
      <canvas ref="canvas" id="preview" :width="containerWidth" :height="containerHeight" class="clip-source"></canvas>
    </div>
  </div>
</template>
<script>
import Cropper from './Cropper';
export default {
  name: 'VueCropper',
  components: {
    Cropper
  },
  props: {
    src: {
      type: String,
      required: true
    },
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
    containerWidth: {
      type: Number,
      default: 600
    },
    containerHeight: {
      type: Number,
      default: 600
    }
  },
  watch: {
    src() {
      this.loadImg();
    }
  },
  data () {
    return {
      img: null,
      imageData: {
        angle: 0,
        direct: false,
        rect: {}
      },
      cropperData: {}
    }
  },
  computed: {
    containerStyle() {
      return {
        width: this.containerWidth + 'px',
        height: this.containerHeight + 'px'
      }
    }
  },
  created() {
    this.loadImg();
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.drawBox = this.$refs.drawBox;
    this.clip = this.$refs.clip;
    // this.rotate();
  },
  methods: {
    // 数据变化
    handleChange() {
      this.$emit('change', this.imageData, this.cropperData);
    },
    // 初始化组件完成
    handleImageInit(data) {
      this.imageData = data;
      this.handleChange();
    },
    handleCropInit(data) {
      this.cropperData = data;
      this.handleChange();
    },
    // 图片完成加载
    loadImg() {
      let image = new Image()
      image.src = this.src;
      image.crossOrigin = "anonymous";
      image.onload = () => {
        // 图片携带的尺寸信息
        // console.log('natural-size:::', this.naturalHeight, this.naturalWidth);
        // const ratio = this.naturalHeight / this.naturalWidth;
        // this.innerHeight = this.width * ratio;
        this.img = image;
        console.log('init data:::', this.imageData, this.cropperData);
        this.paintPicture();
        // this.mountVueCropper();
        // this.drawImage();
      }
    },
    // 图片旋转
    handleImageRotate(data) {
      this.imageData = data;
      this.handleChange();
      this.paintPicture();
    },
    // 绘制裁剪框
    handleCropChange(data) {
      this.cropperData = data;
      this.handleChange();
      this.paintPicture();
    },
    // 计算参数
    getDrawArgs(data) {
      const boxRect = this.drawBox.getBoundingClientRect();
      const { direct, angle, rect } = data;
      let left = rect.left - boxRect.left;
      let top = rect.top - boxRect.top;
      let options = {
        left,
        top,
        ...data
      }
      return options;
    },
    // 完成 合成图片的绘制
    paintPicture() {
      this.clearCanvas();
      const imageData = this.getDrawArgs(this.imageData);
      const cropData = this.getDrawArgs(this.cropperData);
      this.paint(imageData, cropData);
      const { rect = { width: 100, height: 100 } } = cropData;
      const area = {
        width: rect.width + 50,
        height: rect.height + 50,
        x: cropData.left - 20,
        y: cropData.top -20
      };
      this.startClip(area)
    },
    // 绘制图像
    paint(imageData, cropData) {
      const ctx = this.canvas.getContext("2d");
      // this.clearCanvas();
      ctx.save();//保存状态
      const { angle: imgAngle } = imageData;
      const [imgX, imgY, imgWidth, imgHeight] = this.getFinalLeftTop(imageData);
      const { angle: cropAngle } = cropData;
      const [cropX, cropY, cropWidth, cropHeight] = this.getFinalLeftTop(cropData);
      // ctx.translate(canvas.width / 2, canvas.height / 2);//设置画布上的(0,0)位置，也就是旋转的中心点
      ctx.rotate(imgAngle*Math.PI/180);
      // ctx.translate(-canvas.width / 2, -canvas.height / 2);
      ctx.drawImage(this.img, imgX, imgY, imgWidth, imgHeight);//把图片绘制在旋转的中心点，
      ctx.restore();//恢复状态
      ctx.save();
      ctx.globalCompositeOperation = 'destination-in'
      ctx.fillStyle = "green";
      // ctx.translate(canvas.width / 2, canvas.height / 2);//设置画布上的(0,0)位置，也就是旋转的中心点
      ctx.rotate(cropAngle*Math.PI/180);
      ctx.fillRect(cropX, cropY, cropWidth, cropHeight);
      ctx.restore();//恢复状态
    },
    // 裁剪最后的图片
    startClip(area = { width: 100, height: 100 }) {
      const clip = this.clip;
      clip.width = area.width;
      clip.height = area.height;
      const ctx = clip.getContext('2d');
      const imageCtx = this.canvas.getContext("2d");
      const data = imageCtx.getImageData(area.x, area.y, area.width, area.height);

      ctx.putImageData(data, 5, 5);
    },
    // 清空画布
    clearCanvas() {
      const ctx = this.canvas.getContext("2d");
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);  
    },
    // 获取最后绘制元素的左上角顶点坐标
    getFinalLeftTop(options) {
      const { left, top, direct, angle, rect } = options;
      const { width, height } = rect;
      const x2 = left + width / 2;
      const y2 = top + height / 2;
      // 绕中心旋转后的左上角坐标
      const [nx, ny] = this.getLeftTop(x2, y2, left, top, angle, height, width, direct);
      console.log('new cordinate:::', nx, ny);
      // 旋转坐标轴后的坐标
      const [x, y] = this.getNewPos(nx, ny, angle);
      return [x, y, width, height];
    },
    // 计算旋转后的左上角顶点坐标
    /**
     * @params (x2, y2) 旋转中心的坐标
     * @params (x1, y1) 是旋转前顶点坐标
     * @params deg:: 是旋转的角度
     * @params row, col 是容器的宽高
     * @params direct 旋转方向 false: 顺时针 true: 顺时间
     */
    getLeftTop(x2, y2, x1, y1, deg, row, col, direct) {
      x1 = x1; 
      y1 = row - y1; 
      x2 = x2; 
      y2 = row - y2; 
      let angle = Math.PI / 180 * deg;
      let x = (x1 - x2) * Math.cos(angle) + (y1 - y2) * Math.sin(angle) + x2;
      // let diffX = direct ? -(x1 - x2) * Math.sin(angle) : (x1 - x2) * Math.sin(angle)
      let y =  -(x1 - x2) * Math.sin(angle) + (y1 - y2) * Math.cos(angle) + y2;
      x = x;
      y = row - y;
      return [x, y];
    },
    // 旋转画布坐标， 计算新坐标系下的左上角顶点坐标
    getNewPos(x0, y0, deg) {
      let angle = Math.PI / 180 * deg;
      let x = x0 * Math.cos(angle) + y0 * Math.sin(angle);
      let y = y0 * Math.cos(angle) - x0 * Math.sin(angle);
      return [x, y];
    },
    // 下载文件
    handleDownload() {
      const url = this.clip.toDataURL("image/png", 1);
      // document.createElement('a');
      const a = document.createElement('a')
      // a.target = '_blank'
      a.setAttribute('download', 'cropper')
      // a.download = true;
      a.href = url
      document.body.appendChild(a)  
      a.click()
      a.remove()

    }
  }
}
</script>
<style lang="scss">
.vue-cropper__wrap {
  display: flex;
  flex-direction: row;
}

.vue-cropper-container {
  width: 600px;
  height: 600px;
  position: relative;
}
.vue-cropper-preview__wrap {
  position: relative;
  // display: flex;
  // align-items: center;
  // justify-content: center;
}
.clip-source {
  position: fixed;
  top: -9999px;
  visibility: hidden;
}
.buttons {
  display: flex;
  justify-content: flex-start;
  align-items: center;

  .el-button {
    margin-left: 10px;
  }
}
.final-image {
  // position: absolute;
  // border: 1px solid red;
  margin-top: 150px;
  & > #drawer {
    // border: 1px solid blue;
  }
}
</style>
