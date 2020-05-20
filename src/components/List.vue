<template>
  <div class="list-component" ref="list-box">
    <template v-for="(item, i) of tempList" >
      <div
        :ref="i === start + 5 ? 
          'list-top' : i === end -5 ? 
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
    this.handlerVisibled()
  },
  methods:{
    handlerVisibled(){
      const ob = new IntersectionObserver(([entries])=>{
      if(entries.isIntersecting){
        this.start = this.start + 100,
        this.end = this.end + 100
        this.tempList = this.list.slice(this.start, this.end)
        const target = this.$refs['list-box']
        setTimeout(()=>{
          target.scrollTop = 0
          // this.handlerVisibled()
        })
      }
    })
    ob.observe(this.$refs['list-bottom'][0])
    }
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