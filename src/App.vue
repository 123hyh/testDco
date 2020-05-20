<template>
  <div id="app">
    <h4>测试docker容器</h4>
    <List />
    <img alt="Vue logo" src="./assets/logo.png">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import List from '@/components/List.vue'
import axios from 'axios'
export default {
  async created(){
    try {
      const data = await axios.post('/apis/exchangerate/search/settting/',{
        pageIndex: 1,
        pageSize: 10,
        source: "system"
      })
      console.log(data)
    } catch (error) {
      console.log(error)
    }
  },
  name: 'App',
  components: {
    List
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
