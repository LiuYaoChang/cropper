<template>
  <div class="container">
    <div class="box" ref="boxRef" :style="boxStyle" @click.stop="handleFocus">
      <div class="img-box" :style="rotateBoxStyle">
        <!-- <div class="flat">翻转</div> -->
        <div class="cropper-drag-box moveable-wrapper" :style="boxSize">
          <div ref="rotate" class="moveable-rotator" @mousedown="handleMouseDown">
            <svg t="1621676100877" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="1222" width="200" height="200"><path d="M716.8 290.133333c-110.933333-102.4-281.6-106.666667-396.8-12.8S170.666667 537.6 247.466667 665.6c59.733333 106.666667 179.2 166.4 302.933333 149.333333s221.866667-102.4 256-221.866666c8.533333-34.133333 42.666667-51.2 76.8-42.666667 34.133333 8.533333 51.2 42.666667 42.666667 76.8-68.266667 226.133333-302.933333 354.133333-524.8 290.133333C174.933333 853.333333 42.666667 618.666667 106.666667 392.533333c42.666667-145.066667 153.6-256 298.666666-298.666666s298.666667 0 405.333334 102.4l81.066666-81.066667c8.533333-8.533333 21.333333-12.8 34.133334-8.533333 4.266667 12.8 12.8 21.333333 12.8 34.133333v264.533333c0 17.066667-12.8 29.866667-29.866667 29.866667h-260.266667c-12.8 0-25.6-8.533333-29.866666-17.066667s0-25.6 8.533333-34.133333l89.6-93.866667zM512 601.6c-46.933333 0-85.333333-38.4-85.333333-89.6s38.4-89.6 85.333333-89.6 85.333333 38.4 85.333333 89.6-38.4 89.6-85.333333 89.6z" p-id="1223"></path></svg>
          </div>
          <span class="borders" v-if="border">
            <span class="moveable-line line-n" data-dir="n"></span>
            <span class="moveable-line line-s" data-dir="s"></span>
            <span class="moveable-line line-w" data-dir="w"></span>
            <span class="moveable-line line-e" data-dir="e"></span>
          </span>
          <span class="moveable-point point-n" data-dir="n"></span>
          <span class="moveable-point point-e" data-dir="e"></span>
          <span class="moveable-point point-w" data-dir="w"></span>
          <span class="moveable-point point-s" data-dir="s"></span>
          <span class="moveable-point point-ne" data-dir="ne"></span>
          <span class="moveable-point point-se" data-dir="se"></span>
          <span class="moveable-point point-nw" data-dir="nw"></span>
          <span class="moveable-point point-sw" data-dir="sw"></span>
        </div>
        <div v-if="moveAble" :style="boxSize" class="move-target" @mousedown="handleMoveDown" @mousemove="handleMove" @mouseup="handleMoveEnd">
          <!-- <img src="https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1141259048,554497535&fm=26&gp=0.jpg" alt="" class="img"> -->
        </div>
        <div class="drag-content-box" :style="boxSize">
          <slot></slot>
        </div>
        <!-- <div class="rotate1">旋转</div> -->
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'CropperContainer',
  props: {
    width: {
      type: Number,
      default: 100
    },
    height: {
      type: Number,
      default: 100
    },
    moveAble: {
      type: Boolean,
      default: true
    },
    border: {
      type: Boolean,
      default: true
    },
    top: {
      type: [String, Number],
      default: 80
    },
    left: {
      type: [String, Number],
      default: 80
    },
    borderColor: {
      type: String,
      default: '#333333'
    }
  },
  watch: {
    // 调试变化
    height(val) {
      this.register();
    },
    width() {
      this.register();
    }
  },
  data () {
    return {
      boxRef: null,
      rotateRef: null,
      isDown: false, // 鼠标按下
      rotateAngle: 0,
      direction: false, // 记录此次操作的是顺时间针，还是逆时针 // 默认是顺时针
      type: 'rotate', // 标识是否旋转， 还是移动
      mPointB: {
        flag: false,
        x: 0,
        y: 0
      },
      pointStart: {
        x: 0,
        y: 0
      },
      pointEnd: {
        x: 0,
        y: 0
      },
      point: {
        x: 0,
        y: 0
      },
      count: 0,
      end: {
        x: 0,
        y: 0
      },
      oldTarget: {
        target: null,
        angle: 0
      },
      init: {
        count: 0
      }
    }
  },

  computed: {
    hasDefaultBorder() {
      return this.border && !this.focus
    },
    boxSize() {
      return {
        width: this.width + 'px',
        height: this.height + 'px'
      }
    },
    rotateBoxStyle() {
      return {
        transform: `rotate(${this.rotateAngle}deg)`,
        ...this.boxSize
      }
    },
    boxStyle() {
      const { x, y } = this.end;
      if (x > 0 || y > 0) {
        return {
          left: x + 'px',
          top: y + 'px',
          ...this.boxSize
        }
      }
      if (typeof this.left === 'string' && typeof this.top === 'string') {
        return {
          left: this.left,
          top: this.top,
          ...this.boxSize
        }
      }
      return {
        left: this.left + 'px',
        top: this.top + 'px',
        ...this.boxSize
      }
    }
  },
  mounted() {
    // 容器
    this.boxRef = this.$refs.boxRef;
    this.rotateRef = this.$refs.rotate;
    this.getCenterPoint();
    // 注册旋转处理函数
    this.registerMove();
    this.register();
    // this.notifyChange();
  },
  methods: {
    handleFocus() {
      this.focus = true;
    },
    // 初始化完成
    register() {
      let changeData = {
        direct: this.direction,
        angle: this.rotateAngle,
        rect: this.boxRef.getBoundingClientRect(),
        type: this.type
      }
      this.$emit('init', changeData);
    },
    // 获取中心坐标
    getCenterPoint() {
      const boxRect = this.boxRef.getBoundingClientRect();
      let x = boxRect.left + boxRect.width / 2;
      let y = boxRect.top + boxRect.height / 2;
      // 盒子的中心
      this.point.x = x;
      this.point.y = y;
    },
    registerMove() {
      document.addEventListener('mousemove', (e) => {
        e.preventDefault()
        const { pointEnd, pointStart, point, oldTarget } = this;
        let allA = 0;
        if (this.isDown) {
          pointEnd.x = e.clientX;
          pointEnd.y = e.clientY; // 获取结束点坐标
          // 计算每次移动元素的半径变化,用作拉伸

          // 计算出旋转角度
          var AB = {};
          var AC = {};
          AB.x = (pointStart.x - point.x);
          AB.y = (pointStart.y - point.y);
          AC.x = (pointEnd.x - point.x);
          AC.y = (pointEnd.y - point.y); // 分别求出AB,AC的向量坐标表示
          var direct = (AB.x * AC.y) - (AB.y * AC.x); // AB与AC叉乘求出逆时针还是顺时针旋转
          var lengthAB = Math.sqrt(Math.pow(point.x - pointStart.x, 2) +
              Math.pow(point.y - pointStart.y, 2)),
            lengthAC = Math.sqrt(Math.pow(point.x - pointEnd.x, 2) +
              Math.pow(point.y - pointEnd.y, 2)),
            lengthBC = Math.sqrt(Math.pow(pointStart.x - pointEnd.x, 2) +
              Math.pow(pointStart.y - pointEnd.y, 2));
          var cosA = (Math.pow(lengthAB, 2) + Math.pow(lengthAC, 2) - Math.pow(lengthBC, 2)) /
            (2 * lengthAB * lengthAC); //   余弦定理求出旋转角
          var angleA = Math.round(Math.acos(cosA) * 180 / Math.PI);
          if (direct < 0) {
            this.direction = true;
            allA = -angleA; //叉乘结果为负表示逆时针旋转， 逆时针旋转减度数
          } else {
            this.direction = false;
            allA = angleA; //叉乘结果为正表示顺时针旋转，顺时针旋转加度数
          }

          allA += oldTarget.angle
          // $('.img-box').css('transform', 'rotate('+allA+'deg)')

          // 得到旋转角度
          this.rotateAngle = allA;
          // console.log(allA, sc)
          // $('.img-box').css('transform', 'rotate('+allA+'deg)')
        }
      })
      document.addEventListener('mouseup', () => {
        // 把变化 数据上传
        if (this.isDown) {
          this.notifyChange();
        }
        this.isDown = false;
      })
    },
    handleMouseDown(e) {
      e.preventDefault();
      e.stopPropagation();
      this.type = 'rotate';
      const oldTarget = this.oldTarget;
      this.isDown = true;
      const boxRect = this.boxRef.getBoundingClientRect();
      // 计算两个旋转方块之间的角度
      var tanA = boxRect.width / boxRect.height;
      var d = Math.round(Math.atan(tanA) * 180 / Math.PI)
      　　// 如果当前旋转的区域与上一次旋转的区域不一致,需要加上这两个区域之间的角度差2*tanA,这里我只有两个旋转区域,如果有多个区域,需要分别判断
      if (oldTarget.target && oldTarget.target != e.currentTarget) {
        if (e.currentTarget == this.rotateRef) {
          oldTarget.angle = 2 * d
        } else {
          oldTarget.angle = -2 * d
        }
      } else {
        oldTarget.angle = 0
      }
      let pointStart = this.pointStart;
      if (this.count < 1) {
        pointStart.x = e.clientX;
        pointStart.y = e.clientY;
        this.init.count = 0
        oldTarget.target = e.currentTarget // 储存第一次落下的旋转区域
        this.count++
      }
      const mPointB = this.mPointB;
      if (mPointB.flag) { // 如果移动后,元素的B点也需要加上平移的距离
        pointStart.x += mPointB.x
        pointStart.y += mPointB.y
        mPointB.flag = false
        this.init.count = 0
      }
      console.log(5, this.point, pointStart)
    },

    // 移动元素的点击事件
    handleMoveDown(e) {
      this.moveDown = true;
      e.preventDefault()
      e.stopPropagation()
      if (this.init.count < 1) {
        this.init = {
          x: e.clientX,
          y: e.clientY,
          count: 1
        }
      }
      const boxRect = this.boxRef.getBoundingClientRect();
      this.dis = {
        x: e.clientX - boxRect.left,
        y: e.clientY - boxRect.top
      }
    },
    // 移动元素
    handleMove(e) {
      event.preventDefault();
      this.type = 'move';
      // event.stopPropagation()
      if (this.moveDown) {
        const boxRect = this.boxRef.getBoundingClientRect();
        const end = this.end;
        end.x = e.clientX - this.dis.x;
        end.y = e.clientY - this.dis.y;
        // console.log($('.box').offset(), $('.box').position(), end, dis)
        this.point.x = boxRect.width / 2 + boxRect.left;
        this.point.y = boxRect.height / 2 + boxRect.top;
        if (this.count > 0) { // 保存移动的距离
          this.mPointB.x = event.clientX - this.init.x
          this.mPointB.y = event.clientY - this.init.y
          this.mPointB.flag = true
        }
      }
    },
    handleMoveEnd() {
      event.preventDefault();
      if (this.moveDown) {
        this.notifyChange();
      }
      this.moveDown = false;
    },
    notifyChange() {
      let changeData = {
        direct: this.direction,
        angle: this.rotateAngle,
        rect: this.boxRef.getBoundingClientRect(),
        type: this.type
      }
      this.$emit('change', changeData);
    }

  }
}
</script>
<style lang="scss">
  .container {
    /* padding: 20px;
    border: 1px solid sienna; */
    position: relative;
  }
  .box {
    position: absolute;
    left: 200px;
    top: 100px;
    width: 400px;
    height: 400px;
    color: #fff;
    /* background-color: rosybrown; */
  }
  .move-target, .drag-content-box {
    height: 400px;
    width: 400px;
  }
  .move-target {
    position: absolute;
    z-index: 100;
    opacity: 0;
    cursor: move;
  }
  .img-box {
    position: absolute;
    /* left: 30px;
    top: 30px; */
    width: 400px;
    height: 400px;
    background: transparent;
    // background-color: sandybrown;
  }
  .img-box {
    &.is-border {
      border: 2px solid #333333;
    }
  }
  .flat {
    position: absolute;
    right: -20px;
    top: -20px;
    width: 40px;
    height: 40px;
    background-color: seagreen;
    z-index: 3;
    line-height: 40px;
    text-align: center;
    cursor: default;
  }
  .rotate, .rotate1 {
    position: absolute;
    right: -20px;
    bottom: -20px;
    width: 40px;
    height: 40px;
    background-color: royalblue;
    z-index: 3;
    cursor: se-resize;
    line-height: 40px;
    text-align: center;
  }
  .rotate {
    left: 50%;
    top: -20%;
  }
  .rotate1 {
    left: -20px;
    right: auto;
  }
  .img {
    width: 100%;
    height: 100%;
    cursor: move;
  }
  .header {
    height: 50px;
  }

.moveable {

  // &-wrapper {
  //   width: 500px;
  //   height: 500px;
  //   border: 1px solid red ;
  //   position: absolute;
  //   top: 100px ;
  //   left: 100px;
  //   & > span {
  //     display: inline-block;
  //   }
  // }
  &-wrapper {
    display: inline-block;
    position: absolute;
    // z-index: 100;
    width: 100%;
    height: 100%;
  }

  &-box {
    cursor: move;
    width: 100px;
    height: 100px;
    background-color: #ccc;
    position: absolute;
    top: 100px;
    left: 100px;
    box-sizing: border-box;
    & > span {
      display: inline-block;
    }
  }

  &-face,
  &-line,
  &-point {
    display: block;
    height: 100%;
    opacity: 1;
    position: absolute;
    width: 100%;
  }


  &-line {
    background-color: #333333;

    &.line-e {
      cursor: ew-resize;
      right: -3px;
      top: 0;
      width: 2px;
    }

    &.line-n {
      cursor: ns-resize;
      height: 2px;
      left: 0;
      top: -3px;
    }

    &.line-w {
      cursor: ew-resize;
      left: -3px;
      top: 0;
      width: 2px;
    }

    &.line-s {
      bottom: -3px;
      cursor: ns-resize;
      height: 2px;
      left: 0;
    }
  }
  $point-size: 12px;
  $point-position: -($point-size / 2);
  &-point {
    background-color: #333333;
    height: $point-size;
    opacity: 1;
    width: $point-size;
    border-radius: $point-size;

    &.point-e {
      cursor: ew-resize;
      margin-top: $point-position;
      right: $point-position;
      top: 50%;
    }

    &.point-n {
      cursor: ns-resize;
      left: 50%;
      margin-left: $point-position;
      top: $point-position;
    }

    &.point-w {
      cursor: ew-resize;
      left: $point-position;
      margin-top: $point-position;
      top: 50%;
    }

    &.point-s {
      bottom: $point-position;
      cursor: s-resize;
      left: 50%;
      margin-left: $point-position;
    }

    &.point-ne {
      cursor: nesw-resize;
      right: $point-position;
      top: $point-position;
    }

    &.point-nw {
      cursor: nwse-resize;
      left: $point-position;
      top: $point-position;
    }

    &.point-sw {
      bottom: $point-position;
      cursor: nesw-resize;
      left: $point-position;
    }

    &.point-se {
      bottom: $point-position;
      cursor: nwse-resize;
      // height: 20px;
      // opacity: 1;
      right: $point-position;
    }

    // &.point-se {
    //   bottom: -3px;
    //   cursor: nwse-resize;
    //   height: 20px;
    //   opacity: 1;
    //   right: -3px;
    //   width: 20px;

    //   @media (min-width: 768px) {
    //     height: 15px;
    //     width: 15px;
    //   }

    //   @media (min-width: 992px) {
    //     height: 10px;
    //     width: 10px;
    //   }

    //   @media (min-width: 1200px) {
    //     height: 5px;
    //     opacity: 0.75;
    //     width: 5px;
    //   }
    // }

    &.point-se::before {
      background-color: #39f;
      bottom: -50%;
      content: ' ';
      display: block;
      height: 200%;
      opacity: 0;
      position: absolute;
      right: -50%;
      width: 200%;
    }
  }
  $rotate-size: 16px;
  &-rotator {
    // transition: rotate .5s;
    // background-color: #39f;
    margin-left: $point-position;
    opacity: 0.75;
    display: inline-block;
    cursor: pointer;
    position: absolute;
    left: 50%;
    top: -40px;
    width: $rotate-size;
    height: $rotate-size;
    line-height: $rotate-size;
    z-index: 4;
    box-shadow: none;
    border: none;
    transform: translateX(-3px);
    & > .icon {
      font-size: 12px;
      width: $rotate-size;
      height: $rotate-size;
    }
  }
}
</style>

