<template>
  <div id="app">
    <h4>测试docker容器</h4>

    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <List :list="list">
      <template v-slot:header><h2>header</h2></template>
      <template v-slot="row">
        <div>{{ row.data }}</div>
      </template>
    </List>
  </div>
</template>

<script>
import List from '@/components/LongList.vue';
import axios from 'axios';

export default {
  data() {
    return {
      list: [],
    };
  },
  async created() {
    setTimeout(() => {
      this.list = Array.from({ length: 100000 }).map((_, i) => ({ data: i }));
    }, 0);
    try {
      const data = await axios.post('/apis/exchangerate/search/settting/', {
        pageIndex: 1,
        pageSize: 10,
        source: 'system',
      });
      console.log(data);
    } catch (error) {
      console.log(error);
    }
  },
  name: 'App',
  components: {
    List,
  },
};
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
