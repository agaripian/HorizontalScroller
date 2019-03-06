<template>
  <div id="app">
    <!--<Scroller :items="scheduleList">
      <template slot="item" slot-scope="data">
        <img :src="data.item.thumbnail_url" class="js-scroller-control-target" style="height: 142px">
        <h3> <b>{{data.item.position}}</b> : {{data.item.title}} </h3>
      </template>
    </Scroller> -->

      <GridScroller :items="fakeItems">
      <template slot="item" slot-scope="data">
        <!-- need to implement padding bottom hack here -->
        <img :src="`https://via.placeholder.com/808x632/0000FF/FFFFFF?text=${data.index}`" class="js-scroller-control-target" style="width: 100%">
      </template>
    </GridScroller>
  </div>
</template>

<script>
import Scroller from './Scroller.vue';
import GridScroller from './GridScroller.vue';
import schedule from './schedule.js';
export default {
  name: 'app',
  components: {
    Scroller,
    GridScroller,
  },
  data () {
    return {
      scheduleList: schedule.schedule,
      fakeItems: Array(1000).fill({}),
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
