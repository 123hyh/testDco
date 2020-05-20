<template>
  <div class="list-component" ref="list-box" >
    <template v-for="(item, i) of tempList" >
      <div
        :data-serial='item'
        :ref="i === 10 ? 
          'list-top' : i === tempList.length - 10? 
          'list-bottom' : undefined"   
        :key="item"
      >{{item}}</div>
    </template>
    
  </div>
</template>
<script>
/* eslint-disable no-debugger */
export default {
  props:{
    list:{
      type: Array,
      default: ()=>(Array.from({length: 100000}).map((_, i) => i+1))
    }
  },
  mounted(){
    this.handlerBottomVisible()
  },
  methods:{
    // 底部可见时事件
    handlerBottomVisible:(() => {
      let loading = false;
      let handler = null;
      return function() {
      const ob = new IntersectionObserver(([entries])=>{
        if(entries.isIntersecting){

          if ( loading ) return;

          // 打开节流阀
          handler && clearTimeout(handler)
          loading = true

          handler = setTimeout(() => {

            const target = this.$refs['list-box']
            target.scrollTop = target.scrollTop / 2
            this.tempList = this.list.slice(this.start+=100 - 50, this.end+=100 - 50)

            // 关闭节流阀
            loading = false;

            // 取消订阅
            ob.unobserve(entries.target)
            this.$nextTick(this.handlerBottomVisible)
          },150)
        }
      })
      ob.observe(this.$refs['list-bottom'][0])
    }
    })()
  },
  data(){ 
    return  {
    start: 0,
    end: 100,
    tempList: this.list.slice(0, 100)
  }
  }
}
</script>
<style >
.list-component{
  background-color: #f0f0f0;
  margin: 20px;
  height: 300px;
  overflow: auto;
}
</style>