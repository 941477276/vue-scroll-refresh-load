<template>
  <div id="app" class="app">

    <VueScrollRefreshLoad
        ref="vueScroll"
        :pull-up-load-error="pullUpLoadError"
        :pull-up-load-no-more-data="pullUpLoadNoMoreData"
        :height="height"
        @refresh="onRefresh"
        @pullup="onPullup"
        @rePullUp="onRePullUp">
      <template v-if="!loading">
        <h1 v-for="(item, index) in dataList" :key="index">我是第{{item}}个元素</h1>
      </template>
      <h1 v-if="loading">数据加载中...</h1>
    </VueScrollRefreshLoad>
  </div>
</template>

<script>
import VueScrollRefreshLoad from "./components/vue-scroll-refresh-load/VueScrollRefreshLoad";
//import VueScrollRefreshLoad from "../lib/vue-scroll-refresh-load.min.js";
//import VueScrollRefreshLoad from "vue-scroll-refresh-load";

function getOneRandomList(step = 0) {
  console.log('step', step)
  const arr = Array.apply(null, {length: (30 + step)}).map((...args) => {
    return step * 30 + args[1];
  });

  return arr;
}
export default {
  name: "app",
  components: {
    VueScrollRefreshLoad
  },
  data(){
    return {
      dataList: [],
      step: 0,
      pullupTime: 0,
      pullDownTime: 0,
      pullUpLoadError: false,
      pullUpLoadNoMoreData: false,
      height: 300,
      loading: true
    };
  },
  methods: {
    // 触发下拉刷新后的回调事件
    async onRefresh(done) {
      if(this.pullDownTime === 1){
        let timer = setTimeout(() => {
          clearTimeout(timer);
          /*
            不管下拉刷新获取数据成功或失败都需要调用 done() 方法结束当前下拉刷新，
            如不调用 done() 方法，则再次不能进行下拉刷新操作。传递 false 参数表示刷新数据失败
           */
          done(false);
        }, 1500);
        return;
      }
      await this.requestData();
      /*
        不管下拉刷新获取数据成功或失败都需要调用 done() 方法结束当前下拉刷新，
        如不调用 done() 方法，则再次不能进行下拉刷新操作。传递 false 参数表示刷新数据失败
       */
      done();
      this.pullDownTime++;
    },
    // 触发上拉加载后的回调事件
    async onPullup(done){
      if(this.pullupTime === 2){
        let timer = setTimeout(() => {
          clearTimeout(timer);
          this.pullUpLoadError = true;
          //this.pullUpLoadNoMoreData = true;
          /*
            不管上拉加载获取数据成功或失败都需要调用 done() 方法结束当前上拉加载，
            如不调用 done() 方法，则再次不能进行上拉加载操作。
           */
          done();
        }, 1500);
        return;
      }
      await this.requestData(null, true);
      /*
        不管上拉加载获取数据成功或失败都需要调用 done() 方法结束当前上拉加载，
        如不调用 done() 方法，则再次不能进行上拉加载操作。
       */
      done();
      this.pullupTime++;
    },
    async requestData(cb, isAdditional) {
      try {
        const newData = await this.ajaxGet(/* url */);
        this.loading = false;
        if(isAdditional){
          let arr = this.dataList.concat(newData);
          this.dataList = arr;
        }else {
          this.dataList = newData;
        }

        this.step++;
        if(cb && typeof cb === 'function'){
          cb();
        }
      } catch (err) {
        console.log(err);
      }
    },
    ajaxGet(/* url */) {
      return new Promise(resolve => {
        let timer = setTimeout(() => {
          clearTimeout(timer);
          const dataList = getOneRandomList(this.step);
          resolve(dataList);
        }, 2000);
      });
    },
    // 重新加载数据事件，点击重新加载按钮后立即触发
    async onRePullUp(done){
      this.pullUpLoadError = false;
      await this.requestData(null, true);
      done();
      this.pullupTime++;
    }
  },
  mounted() {
    this.requestData(() => {
      // 获取数据后需要刷新 better-scroll ，否则不能滚动或滚动不正确
        this.$refs.vueScroll.refresh();

        let timer = setTimeout(() => {
        /*console.log('手动下拉刷新！');
        this.$refs.vueScroll.autoPullDownRefresh();*/
        console.log('改变高度了');
        clearTimeout(timer);
        // 动态改变高度
        this.height = 450;
      }, 1000);
    });
  }
};
</script>

<style lang="less">
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
/*.app{
  height: 100vh;
}*/
</style>
