<template>
  <div class="moveBox" @mouseenter="timeOut" @mouseleave="timeSuccess">
    <!-- 左箭头 -->
    <div
      class="leftClick"
      @click="leftClick"
      v-if="isShow ? !isShow : !isHidden"
    >
      <img src="./images/leftClick.png" alt="" draggable="false" />
    </div>
    <!-- 右箭头 -->
    <div
      class="rightClick"
      @click="rightClick"
      v-if="isShow ? !isShow : !isHidden"
    >
      <img src="./images/rightClick.png" alt="" draggable="false" />
    </div>
    <!-- 轮播图 -->
    <ul class="moveBoxImg" ref="moveBoxImg" :style="`width:${ulWidth}px;`">
      <!-- 渲染结构 -->
      <li v-for="(item, index) in pictureData" :key="index">
        <a :href="item.href || 'javascript:;'" class="default">
          <img
            :src="item.url"
            alt=""
            draggable="false"
            :style="`width:${moveBxoWidth}px;height: ${moveBxoHeight}px;`"
          />
        </a>
      </li>
      <!-- 复制最前面的那一张，实现无缝滚动的效果 -->
      <li>
        <a :href="imgUrl[0].href" class="default">
          <img
            :src="imgUrl[0].url"
            alt=""
            :style="`width:${moveBxoWidth}px;height: ${moveBxoHeight}px;`"
          />
        </a>
      </li>
    </ul>
    <!-- 小圆点的索引 -->
    <ol class="moveCircle" v-if="isShow ? !isShow : !circle">
      <li
        v-for="(item, index) in imgUrl"
        :key="index"
        :class="imgCircle == index ? 'active' : ''"
        @click="circleClick(index)"
      ></li>
    </ol>
  </div>
</template>

<script>
export default {
  name: "MoveBox",
  props: ["imgUrl", "circle", "time", "isLR", "state"],
  data() {
    return {
      moveBxoWidth: "", // 盒子的宽度
      moveBxoHeight: "", // 盒子的高度
      ulWidth: "", //图片总盒子的宽度
      pictureData: [], //图片总数据
      nextPicture: 0, //当前图片的索引
      imgCircle: 0, //小圆点的索引
      isThrottle: true, // 节流
      isHidden: "", //是否隐藏左右按钮
    };
  },
  created() {
    // 动画函数封装
    this.animate = (obj, end, callback) => {
      clearInterval(obj.timer);
      obj.timer = setInterval(() => {
        let step = (end - obj.offsetLeft) / 10;
        step = step > 0 ? Math.ceil(step) : Math.floor(step);
        if (obj.offsetLeft == end) {
          callback && callback();
          clearInterval(obj.timer);
          return;
        }
        obj.style.left = obj.offsetLeft + step + "px";
      }, 10);
    };
  },
  // 生命周期函数
  mounted() {
    // 把图片数据给pictureData
    this.pictureData = this.imgUrl;
    // 把盒子的宽度给moveBxoWidth属性进行计算
    this.moveBxoWidth = this.$refs.moveBoxImg.offsetWidth;
    // 把盒子的宽度给moveBxoWidth属性进行计算
    this.moveBxoHeight = this.$refs.moveBoxImg.offsetHeight;
    // 计算外层盒子的宽度
    this.ulWidth = this.moveBxoWidth * (this.pictureData.length + 1);
    // 页面数据渲染完成以及数据挂载完成，开启一次定时器，让轮播图进行播放
    this.timer = setInterval(() => {
      if (this.imgUrl.length != 1) {
        this.rightClick();
      }
    }, this.time || 2500);
    // 把是否需要隐藏左右按钮的布尔值赋值给isHidden
    this.isHidden = this.isLR;
    // 判断用户否隐藏左右按钮。如果隐藏，则鼠标滑过的说有事件则不生效
    if (!this.isLR && this.state) {
      this.isHidden = true;
    }
  },
  methods: {
    // 左击
    leftClick() {
      if (this.isThrottle) {
        this.isThrottle = false;
        this.nextPicture <= 0
          ? ((this.nextPicture = this.pictureData.length),
            (this.$refs.moveBoxImg.style.left =
              -this.nextPicture * this.moveBxoWidth + "px"))
          : (this.nextPicture = this.nextPicture);
        // 每次点击索引减一
        this.nextPicture--;
        // 计算出要移动的距离
        let index = -this.nextPicture * this.moveBxoWidth;
        // 调用动画函数
        this.animate(this.$refs.moveBoxImg, index, () => {
          this.isThrottle = true;
        });
        // 实现小圆点的变化效果
        this.imgCircle--;
        this.imgCircle < 0
          ? (this.imgCircle = this.pictureData.length - 1)
          : this.imgCircle;
      }
    },
    // 右击
    rightClick() {
      if (this.isThrottle) {
        this.isThrottle = false;
        // 每次点击索引加一
        this.nextPicture++;
        this.nextPicture =
          this.nextPicture == this.pictureData.length + 1
            ? ((this.$refs.moveBoxImg.style.left = 0), (this.nextPicture = 1))
            : this.nextPicture;
        // 计算出要移动的距离
        let index = -this.nextPicture * this.moveBxoWidth;
        // 调用动画函数
        this.animate(this.$refs.moveBoxImg, index, () => {
          this.isThrottle = true;
        });
        // 实现小圆点的变化效果
        this.imgCircle++;
        this.imgCircle > this.pictureData.length - 1
          ? (this.imgCircle = 0)
          : this.imgCircle;
      }
    },
    // 点击小圆点也能切换轮播图片
    circleClick(id) {
      // 监视小圆点的索引
      this.imgCircle = id;
      // 把当前小圆点的索引给nextPicture进行实时绑定
      this.nextPicture = id;
      // 调用动画函数
      this.animate(this.$refs.moveBoxImg, -id * this.moveBxoWidth);
    },
    // 关闭定时器
    timeOut() {
      clearInterval(this.timer);
      this.timer = null;
      // 判断用户否隐藏左右按钮。如果隐藏，则鼠标滑过的说有事件则不生效
      if (!this.isLR && this.state) {
        this.isHidden = false;
      }
    },
    // 开启定时器
    timeSuccess() {
      this.timer = setInterval(() => {
        if (this.imgUrl.length != 1) {
          this.rightClick();
        }
      }, this.time || 2500);
      // 判断用户否隐藏左右按钮。如果隐藏，则鼠标滑过的说有事件则不生效
      if (!this.isLR && this.state) {
        this.isHidden = true;
      }
    },
  },
  // 计算属性
  computed: {
    isShow() {
      return this.imgUrl.length == 1;
    },
  },
  // 在组件销毁之前清除定时器
  beforeDestroy() {
    this.animate = null;
    clearInterval(this.timer);
  },
};
</script>

<style lang="less" scoped>
ul,
ol {
  padding: 0;
  margin: 0;
  list-style: none;
}
img {
  vertical-align: middle;
}
.default {
  cursor: default;
}
.moveBox {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  //   左箭头
  .leftClick,
  .rightClick {
    display: flex;
    justify-content: center;
    width: 8%;
    text-align: center;
    padding: 2% 0;
    cursor: pointer;
    background: rgba(0, 0, 0, 0.3);
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    z-index: 2;
    img {
      width: 60%;
    }
  }
  .leftClick {
    left: 0;
    border-top-right-radius: 10px;
    border-bottom-right-radius: 10px;
  }
  .rightClick {
    right: 0;
    border-top-left-radius: 10px;
    border-bottom-left-radius: 10px;
  }

  //   图片
  .moveBoxImg {
    position: absolute;
    left: 0;
    width: 100%;
    height: 100%;
    // z-index: 2;
    li {
      float: left;
      img {
        width: 100%;
        height: 100%;
      }
    }
  }
  .moveCircle {
    position: absolute;
    bottom: 3%;
    width: 100%;
    text-align: center;
    li {
      display: inline-block;
      padding: 1%;
      background: #fff;
      border-radius: 50%;
      margin-left: 1%;
      cursor: pointer;
    }
    li.active {
      background: #000;
    }
  }
}
</style>