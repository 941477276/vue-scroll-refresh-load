# vue-scroll-refresh-load
A better-scroll - based drop-down refresh, pull-up load Vue plug-in(一个基于better-scroll的下拉刷新、上拉加载Vue插件)
# 效果预览
![效果预览](./src/assets/vueScrollRefreshLoad_effect.gif)

[Live Demo](https://941477276.github.io/vue-scroll-refresh-load/dist/index.html)

# 安装
`npm install vue-scroll-refresh-load --save`
# 基本使用
```
/*****main.js****/
import VueScrollRefreshLoad from 'vue-scroll-refresh-load';
// 全局注册组件
Vue.component(VueScrollRefreshLoad.name, VueScrollRefreshLoad);



/*****组件内使用****/
<template>   
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
</template>

<script>
import VueScrollRefreshLoad from "./components/vue-scroll-refresh-load/VueScrollRefreshLoad";

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
    // 触发下拉刷新后的回调事件 ---- 必要的
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
    // 触发上拉加载后的回调事件 ---- 必要的
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
```
# 组件`Props`
| Props名称 | 描述 | 数据类型 | 默认值 |
| :-------- | :--- | :------- | :----- |
| `loadingText` | 加载中提示文字，下拉刷新、上拉加载的都会使用到 | `Boolean` | 加载中... |
| `beforePullDownText` | 下拉刷新前显示的文字 | `String` | 下拉即可刷新 |
| `pullDownRefreshSuccessText` | 下拉刷新成功获取数据后的文字 | `String` | 数据刷新成功！ |
| `pullDownRefreshErrorText` | 下拉刷新获取数据失败后的文字 | `String` | 数据加载失败！ |
| `pullUpFetchDataErrorText` | 上拉加载数据请求失败时显示的文字 | `String` | 请求失败，点击重新加载 |
| `pullUpFetchDataNoMoreText` | 上拉加载数据无更多数据时显示的文字 | `String` | 已经到底部，没有更多内容了 |
| `pullDownConfig` | 下拉刷新配置，如果配置值为`false`，则不会开启下拉刷新功能。具体配置请参考 [better-scroll pulldown配置](https://better-scroll.github.io/docs/zh-CN/plugins/pulldown.html) | `Boolean`、`Object` | `{ threshold: 60 }` |
| `pullUpConfig` | 上拉加载配置，如果配置值为`false`，则不会开启上拉加载功能。具体配置请参考 [better-scroll pullup配置](https://better-scroll.github.io/docs/zh-CN/plugins/pullup.html) | `Boolean`、`Object` | `true` |
| `betterScrollConfig` | better-scroll配置。具体配置请参考 [better-scroll配置](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-options.html) | `Object` | `{}` |
| `scrollbar` | 是否开启scrollbar | `Boolean` | `true` |
| `height` | 容器高度，如果容器高度为0则默认取父级容器的高度。高度变化会自动刷新better-scroll | `Number` | `0` |
| `stopTime` | 数据刷新后better-scroll弹起停留时间 | `Number` | `600` |
| `pullUpLoadError` | 上拉加载数据是否出错，为`true`底部会显示数据加载出错提示，点击可重新获取数据 | `Boolean` | `false` |
| `pullUpLoadNoMoreData` | 上拉加载是否已经没有更多数据了 | `Boolean` | `false` |
| `click` | 是否支持click事件，与better-scroll配置click作用一致 | `Boolean` | `true` |
| `autoInit` | 是否自动初始化 better-scroll | `Boolean` | `true` |
| `disabled` | 是否禁用 better-scroll | `Boolean` | 'false' | 
| `dataLoading` | 数据是否加载中，每当这个值改变，better-scroll都会自动刷新 | `Boolean` | 'false' | 

# 组件事件
| 事件名 | 描述 | 回调参数 |
| :-------- | :--- | :---- |
| `onInit` | better-scroll 初始化完成事件 | better-scroll初始化完成后会立即触发该事件 |
| `refresh` | 下拉刷新事件 | `done方法`不管下拉刷新数据成功或失败都必须调用`done`方法来结束下拉刷新，<br>否则不能再次进行下拉刷新操作。`done`方法还可以传递一个`Boolean`类型的参数，<br>当传递的参数`false`表示下拉刷新失败，下拉刷新失败后会显示下拉刷新失败后的提示文案！ |
| `pullup` | 上拉加载事件 | `done方法`不管下拉刷新数据成功或失败都必须调用`done`方法来结束上拉加载，<br>否则不能再次进行上拉加载操作。 |
| `rePullUp` | 点击按钮重新上拉加载事件 | `done方法`不管下拉刷新数据成功或失败都必须调用`done`方法来结束上拉加载，<br>否则不能再次进行上拉加载操作。 |

# 可用方法
| 方法名 | 描述 | 参数 | 返回值 |
| :-------- | :--- | :--- | :---- |
| `initScroll` | 初始化better-scroll | 无 | `better-scroll`实例 |
| `refresh` | 刷新better-scroll | 无 | `undefined` |
| `autoPullDownRefresh` | 手动执行下拉刷新操作 | `isLoadDataSuccess`表示下拉刷新数据是否成功，<br>数据类型为`Boolean` | `undefined` |
| `finishPullDown` | 结束下拉刷新 | 无 | `undefined` |
| `finishPullUp` | 结束上拉加载 | 无 | `undefined` |
| `enable` | 启用better-scroll | 无 | `undefined` |
| `disable` | 禁用better-scroll | 无 | `undefined` |
| `scrollTo` | 同better-scroll `scrollTo`方法 | `x`, `y`, `time`, `easing` | `undefined` |
| `scrollBy` | 同better-scroll `scrollBy`方法 | `x`, `y`, `time`, `easing` | `undefined` |
| `scrollToElement` | 同better-scroll `scrollToElement`方法 | `el`, `time`, `offsetX`, `offsetY`, `easing` | `undefined` |
