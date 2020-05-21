<template>
  <div class="xy-long-list-box" >
    <slot name='header'></slot>
    <div ref="list-box" class="long-list-content">
      <!-- 支持容器顶部 -->
        <template v-for="(item, i) of tempList">
          <div
            :ref="
              i === 10
                ? 'list-top'
                : i === tempList.length - 10
                ? 'list-bottom'
                : undefined
            "
            :key="item.key"
          >
          <slot name="content" :current='item'></slot>
          </div>
        </template>
    </div>
  </div>
</template>
<script>
/* eslint-disable no-debugger */
/**
 * 初始化长列表
 * @param {Function} cb 观察的元素可见时调用该方法
 * @param {Object} options 
 * @param {String} options.refStr 需要订阅的 ref 的 name
 * 
 */
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

            // 节流阀控制
            if (loading) {
              return;
            }
            // 取消订阅当前可见的元素
            observer.unobserve(entries.target);
            this.$nextTick(()=>{

            // 打开节流阀
            handler && clearTimeout(handler);
            loading = true;

            handler = setTimeout(() => {
              const target = this.$refs['list-box'];
              
              if (
                (refStr === 'list-bottom' &&
                  this.end > this.list.length) ||
                (refStr === 'list-top' &&
                  this.start !== 0 &&
                  this.start - this.pageSize < 0)
              ) {

                // 防止某一处到达最大值时，事件的消失
                this.$nextTick(()=>{

                  // 关闭节流阀
                  loading = false;
                  if(refStr === 'list-top' ){
                    // 接触到顶部是需要重置start 和 end
                    this.start = 0;
                    this.end = 100
                    this.handlerBottomVisible() 
                  }else{
                    this.handlerScrollTopEvent()
                  }
                })
                return;
              }

              // 先清空列表
              this.tempList = []
              // 执行回调函数
              cb.call(this, entries.target);
              target.scrollTop = target.scrollTop / 2;

              // 关闭节流阀
              loading = false;

              this.$nextTick(() => {
                this.handlerScrollTopEvent();
                this.handlerBottomVisible();
              });
            }, 150);
            })
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
      default: () => ([]),
    },
  },
  mounted() {
    this.handlerBottomVisible();
  },
  methods: {
    // 顶部可见时事件
    handlerScrollTopEvent: initLongList(
      function() {
        console.log(`触发顶部`)
        const start = this.start-=this.pageSize
        const end = this.end-=this.pageSize
        const calc = this.pageSize / 2 - 10;
        
        const params = [
          start <= 0 ? 0 : start - calc,
          end <= 100 ? 100 : end - calc
        ]
        this.tempList = this.list.slice(...params).map((item,i)=>{
          item.key = i
          return item
        });
      },
      { refStr: 'list-top' }
    ),
    handlerBottomVisible: initLongList(
      function() {
        const start = (this.start += this.pageSize);
        const end = (this.end += this.pageSize);
        const calc = this.pageSize / 2 - 10;
        this.tempList = this.list.slice(start - calc, end - calc).map((item,i)=>{
          item.key = i
          return item
        });
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
.xy-long-list-box .long-list-content {
  margin: 20px;
  height: 600px;
  overflow: auto;
  border: 1px solid;
}
</style>
