<template>
  <div class="long-list-box">
    <!-- 头部 -->
    <slot name="header"></slot>
    <div class="long-list-container" ref='container'>
      <!-- 支撑顶部容器高度 -->
      <div :style="topContainer" ></div>

      <!-- 中间元素 -->
      <template v-for="item in currentList">
        <div :key="item" ref="container-item">{{item}}</div>
      </template>

      <!-- 支持底部容器高度 -->
      <div :style="bottomContainer" ></div>
    </div>
  </div>
</template>

<script>
/* eslint-disable no-debugger */
export default {
  props: {
    list:{
      type: Array,
      default: ()=> Array.from({length: 500}).map((_,i)=>i)
    }
  },
  data(){
    return {
      topContainer:{
        height: '0px'
      },
      bottomContainer:{
        height: '0px'
      },
      currentList: this.list.slice(0, 100),
      currentListHeight: 0
    }
  },
  watch:{
    list:{
      handler(){

      },
      deep: true
    }
  },
  mounted(){
    const refs = this.$refs['container-item'] && this.$refs['container-item'][0];
    // 计算容器内 item 的总高度
    if(refs){
      const x  = window.getComputedStyle(refs).height
      const y = +x.match(/^\d+/)[0]
      const height = this.currentListHeight = this.currentList.length * y
      this.topContainer.height = (height / 2) + 'px'
      this.bottomContainer.height = (height / 2) + 'px';
      this.$nextTick(()=>{
        this.$refs.container.scrollTop = height / 2
      })
    }
  }
}
</script>

<style>
.long-list-box{
  width: 100%;
  background: #f0f0f0;
}
.long-list-box > .long-list-container{
  width: 100%;
  height: 600px;
  overflow: auto;
}
</style>