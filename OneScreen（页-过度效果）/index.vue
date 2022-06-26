<template>
  <div class="containerBox" ref="boxHeight">
    <div class="bigBox" ref="bigBox" :style="`height: ${bigHeight}px`">
      <div
        class="samllBox"
        :style="`height: ${childrensHeight}px`"
        v-for="(item, index) in pages"
        :key="index"
      >
        <slot :name="`page${item}`"></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "OneScreen",
  props: ["page"],
  data() {
    return {
      childrensHeight: null, // 当前窗口的宽度
      childrenSum: null, // 子元素的数量
      index: 0, // 当前的索引值
      flag: true, // 节流阀
    };
  },
  created() {
    this.animate = function (obj, target, callback) {
      clearInterval(obj.timer);
      obj.timer = setInterval(() => {
        let step = (target - obj.offsetTop) / 12;
        step = step > 0 ? Math.ceil(step) : Math.floor(step);
        if (target == obj.offsetTop) {
          callback && callback();
          clearInterval(obj.timer);
          return;
        }
        obj.style.top = obj.offsetTop + step + "px";
      }, 10);
    };
  },
  mounted() {
    // 获取当前屏幕的宽度
    this.childrensHeight = this.$refs.boxHeight.offsetHeight;
    // 获取子元素的数量
    this.childrenSum = this.$refs.bigBox.childElementCount;
    window.onresize = () => {
      // 获取当前屏幕的宽度
      this.childrensHeight = this.$refs.boxHeight.offsetHeight;
      // 获取子元素的数量
      this.childrenSum = this.$refs.bigBox.childElementCount;
      // 调用动画函数，进行移动
      this.animate(this.$refs.bigBox, -this.childrensHeight * this.index);
    };

    window.onwheel = (e) => {
      if (this.flag) {
        // 关闭节流阀
        this.flag = false;
        // 判断滑轮往上滚动还是往下滚动
        if (e.wheelDeltaY > 0) {
          this.index--;
          this.index = this.index < 0 ? "0" : this.index;
        } else if (e.wheelDeltaY < 0) {
          this.index++;
          if (this.index > this.childrenSum - 1) {
            this.index = 0;
          }
        }
        // 调用动画函数，进行移动
        this.animate(
          this.$refs.bigBox,
          -this.childrensHeight * this.index - 40,
          () => {
            let element = this.$refs.bigBox;
            clearInterval(element.times);
            element.times = setInterval(() => {
              let step =
                (-this.childrensHeight * this.index - element.offsetTop) / 50;
              step = step > 0 ? Math.ceil(step) : Math.floor(step);
              if (-this.childrensHeight * this.index == element.offsetTop) {
                clearInterval(element.times);
                this.flag = true;
              }
              element.style.top = element.offsetTop + step + "px";
            });
          }
        );
      }
    };
  },
  computed: {
    bigHeight() {
      return this.childrensHeight * this.childrenSum;
    },
    pages() {
      let arr = [];
      for (let i = 1; i <= this.page; i++) {
        arr.push(i);
      }
      return arr;
    },
  },
};
</script>

<style lang="less" scoped>
* {
  margin: 0;
  padding: 0;
}
html,
body {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
.containerBox {
  width: 100%;
  height: 100%;
  background: #ccc;
  .bigBox {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    .samllBox {
      width: 100%;
    }
  }
}
</style>