<template>
  <div class="app-container class-name app-page">
    <div class="vue-cropper-wrapper">
      <vue-cropper
        ref="vueCrop"
        @change="handleDataChange"
        :src="imgSrc"
        :container-height="500">
        <el-upload
          class="upload-demo"
          slot="button"
          :show-file-list="false"
          action="#"
          :auto-upload="false"
          :on-change="handleFileChange">
          <el-button size="small" type="primary">上传文件</el-button>
        </el-upload>
      </vue-cropper>
    </div>
    <div class="cropper-args__wrap">
      <el-row :gutter="20">
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">widthA</label>
            <el-input v-model="contentData.width"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">heightA</label>
            <el-input v-model="contentData.height"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">angleA</label>
            <el-input v-model="contentData.angle"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">xA</label>
            <el-input v-model="contentData.x"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">yA</label>
            <el-input v-model="contentData.y"></el-input>
          </div>
        </el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">widthB</label>
            <el-input v-model="cropData.width"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">heightB</label>
            <el-input v-model="cropData.height"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">angleB</label>
            <el-input v-model="cropData.angle"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">xB</label>
            <el-input v-model="cropData.x"></el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <div class="cropper-args__item">
            <label for="">yB</label>
            <el-input v-model="cropData.y"></el-input>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>
<script>
import VueCropper from './components/Cropper/index';
export default {
  components: {
    VueCropper
  },
  data () {
    return {
      input: 0,
      img: null,
      contentData: {
        width: 0,
        height: 0,
        angle: 0,
        x: 0,
        y: 0
      },
      cropData: {
        width: 0,
        height: 0,
        angle: 0,
        x: 0,
        y: 0
      },
      imgSrc: 'https://cubemall-product-dazzle.oss-cn-beijing.aliyuncs.com/user-dir-prefix/bd264fd7-4cd5-49bf-8362-0d7f865773d4_a1avjj30098892614268.jpeg'
    }
  },
  mounted() {},
  methods: {
    // 配置数据变化
    handleDataChange(imageData, cropData) {
      console.log('data change:::', imageData, cropData)
      if (imageData.rect) {
        this.updateImageData(imageData);
      }
      if (cropData.rect) {
        this.updateCropData(cropData);
      }
    },
    // 获取本地文件
    handleFileChange(file) {
      console.log('file::', file)
      this.getImgSrc(file.raw).then(url => {
        if (url) {
          this.imgSrc = url;
        }
      });
    },
    getImgSrc(file) {
      let objUrl;
      return new Promise((resolve) => {
        if (window.FileReader) {
          let reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onloadend = function (e) {
            objUrl = e.target.result; // 获取到的src可以使用的信息
            resolve(objUrl);
          }
        } else {
          resolve(null); 
        }
      })
    },
    updateImageData(imageData) {
      const { angle, rect } = imageData;
      this.contentData.width = rect.width;
      this.contentData.height = rect.height;
      this.contentData.angle = angle;
      this.contentData.x = rect.x;
      this.contentData.y = rect.y;
    },
    updateCropData(cropData) {
      const { angle, rect } = cropData;
      this.cropData.width = rect.width;
      this.cropData.height = rect.height;
      this.cropData.angle = angle;
      this.cropData.x = rect.x;
      this.cropData.y = rect.y;
    }
  }
}
</script>
<style lang="scss">
  * {
    margin: 0;
    padding: 0
  }
  body, html {
    height: 100%;
  }

  .cropper-args__wrap {
    padding: 20px;
  }

  .cropper-args__item {
    display: flex;
    justify-content: flex-start;
    align-items: center;

    & .el-input {
      margin-left: 10px;
    }
  }
  .el-row + .el-row {
    margin-top: 10px;
  }
</style>

