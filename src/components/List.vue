<template>
  <div class="list-component" ref="list-box">
    <template v-for="(item, i) of tempList">
      <div
        :data-serial="item"
        :ref="
          i === 10
            ? 'list-top'
            : i === tempList.length - 10
            ? 'list-bottom'
            : undefined
        "
        :key="item"
      >
        {{ item }}
      </div>
    </template>
  </div>
</template>
<script>
/* eslint-disable no-debugger */

// 初始化长列表
function initLongList(cb = () => {}, { refStr } = {}) {
  let loading = false;
  let handler = null;
  let observer = null;
  return function() {
    // 判断是否要处理继续往下处理逻辑
    if (
      this.$refs[refStr] &&
      Object.prototype.toString.call(this.$refs[refStr][0]).slice(8, -1) ===
        'HTMLDivElement'
    ) {
      // 单例
      observer =
        observer ||
        new IntersectionObserver(([entries]) => {
          if (entries.isIntersecting) {
            if (loading) {
              return;
            }
            // 取消订阅上一个
            observer.unobserve(entries.target);

            // 打开节流阀
            handler && clearTimeout(handler);
            loading = true;

            handler = setTimeout(() => {
              const target = this.$refs['list-box'];
              target.scrollTop = target.scrollTop / 2;
              if (
                (refStr === 'list-bottom' &&
                  this.start + this.pageSize >= this.list.length) ||
                (refStr === 'list-top' &&
                  this.start !== 0 &&
                  this.start - this.pageSize <= 0)
              ) {
                return;
              }
              // 执行回调函数
              cb.call(this, entries.target);

              // 关闭节流阀
              loading = false;

              this.$nextTick(() => {
                this.handlerScrollTopEvent();
                this.handlerBottomVisible();
              });
            }, 70);
          }
        });
      observer.observe(this.$refs[refStr][0]);
    } else {
      console.warn(
        `数据量 < props.pageSize \${${this.pageSize}} ${this.pageSize}, 不进行长列表优化~`
      );
    }
  };
}
export default {
  props: {
    /* 容器中的条目数 */
    pageSize: {
      type: Number,
      default: 50,
    },
    /* 源列表数据 */
    list: {
      type: Array,
      default: () => Array.from({ length: 500 }).map((_, i) => i + 1),
    },
  },
  mounted() {
    this.handlerBottomVisible();
  },
  methods: {
    // 顶部可见时事件
    handlerScrollTopEvent: initLongList(
      function() {
        console.log(1);
      },
      { refStr: 'list-top' }
    ),
    handlerBottomVisible: initLongList(
      function() {
        const start = (this.start += this.pageSize);
        const end = (this.end += this.pageSize);
        const calc = this.pageSize / 2;
        this.tempList = this.list.slice(start - calc, end - calc);
      },
      { refStr: 'list-bottom' }
    ),
  },
  data() {
    return {
      start: 0,
      end: 100,
      tempList: this.list.slice(0, 100),
    };
  },
};
</script>
<style>
.list-component {
  background-color: #f0f0f0;
  margin: 20px;
  height: 300px;
  overflow: auto;
}
</style>
