<template>
  <div id="app">
    <Scroller :items="scheduleList">
      <template slot="item" slot-scope="data">
        <!-- need to implement padding bottom hack here -->
        <img :src="data.item.thumbnail_url" class="js-scroller-control-target" style="height: 142px">
        <h3> <b>{{data.item.position}}</b> : {{data.item.title}} </h3>
      </template>
    </Scroller>
  </div>
</template>

<script>
import Scroller from './Scroller.vue';
import schedule from './schedule.js';
export default {
  name: 'app',
  components: {
    Scroller,
  },
  data () {
    return {
      scheduleList: schedule.schedule
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}
</style>
