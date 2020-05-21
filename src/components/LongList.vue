<template>
  <div class="long-list-box">
    <!-- 头部 -->
    <slot name="header"></slot>
    <div class="long-list-container" ref="container" @scroll="handlerScroll">
      <!-- 中间元素 -->
      <template v-for="item in currentList">
        <div :key="item" :ref="item | transformRef(currentList)">
          {{ item }}
        </div>
      </template>

      <!-- 支持底部容器高度 -->
      <div :style="bottomContainer" ref="container-bottom-box"></div>
    </div>
  </div>
</template>

<script>
/* eslint-disable no-debugger */
export default {
  filters: {
    transformRef(item, val) {
      if (item === 1) {
        return 'container-first-item';
      } else if (item === val.length - 1) {
        return 'container-last-item';
      }
    },
  },
  props: {
    list: {
      type: Array,
      default: () => Array.from({ length: 1000 }).map((_, i) => i),
    },
  },
  data() {
    return {
      pageIndex: 1,
      itemHeight: 0,
      bottomContainer: {
        height: '0px',
      },
      currentList: this.list.slice(0, 100),
    };
  },
  watch: {
    list: {
      handler() {},
      deep: true,
    },
  },
  computed: {
    // 当前item的总高度
    currentListHeight() {
      return this.currentList.length * this.itemHeight || 0;
    },
    // 容器的总高度
    totalHeight() {
      return this.list.length * this.itemHeight || 0;
    },
  },
  mounted() {
    const refs =
      this.$refs['container-first-item'] &&
      this.$refs['container-first-item'][0];
    // 计算容器内 item 的总高度
    if (refs) {
      this.itemHeight = this.handlerCalcHeight(
        this.$refs['container-first-item'][0]
      );
      const transNumber = this.handlerCalcHeight(refs);
      const height = this.currentList.length * transNumber;
      this.bottomContainer.height =
        this.list.length * transNumber - height + 'px';
      this.$nextTick(this.handlerBottom.bind(this));
    }
  },
  methods: {
    handlerScroll() {},
    // 计算高度
    handlerCalcHeight(target) {
      const itemHeight = window.getComputedStyle(target).height;
      return +itemHeight.match(/^\d+/)[0];
    },
    handlerBottom: (() => {
      let observe = null;
      return function() {
        observe =
          observe ||
          new IntersectionObserver(([entris]) => {
            if (entris.isIntersecting) {
              this.pageIndex += 1;
              this.currentList.push(
                ...this.list.slice(
                  (this.pageIndex - 1) * 100,
                  this.pageIndex * 100
                )
              );

              // 减去底部盒子的高度
              this.bottomContainer.height =
                this.totalHeight - this.currentListHeight + 'px';
            }
          });
        observe.observe(this.$refs['container-bottom-box']);
      };
    })(),
  },
};
</script>

<style>
.long-list-box {
  width: 100%;
  background: #f0f0f0;
}
.long-list-box > .long-list-container {
  width: 100%;
  height: 600px;
  overflow: auto;
}
</style>
