<template>
  <section class="vue-scroll-refresh-load" ref="vueScrollRefreshLoad">
    <div class="vue-scroll-wrapper" ref="bsWrapper">
      <div class="vue-scroll-content">
        <!--下拉刷新提示 start-->
        <div class="pulldown-tip" v-if="pullDownConfig">
          <slot name="pulldownTip">
            <slot name="pullDownLoading">
              <div class="scroll-loading-wrap" v-show="pulldownData.loading">
                <div class="scroll-loading">
                  <svg viewBox="25 25 50 50" class="loading-circular">
                    <circle cx="50" cy="50" r="20" fill="none"></circle>
                  </svg>
                </div>
                <span class="scroll-loading-text"><!--加载中...-->{{loadingText}}</span>
              </div>
            </slot>

            <div class="pulldown-text" v-show="pulldownData.beforePullDown" v-html="beforePullDownText"><!--下拉即可刷新--></div>
            <div v-show="pulldownData.refreshDataSuccess" v-html="pullDownRefreshSuccessText"><!--数据刷新成功！--></div>
            <div v-show="pulldownData.refreshDataError" v-html="pullDownRefreshErrorText"><!--数据加载失败！--></div>
          </slot>
        </div>
        <!--下拉刷新提示 end-->

        <div class="slot-content">
          <slot></slot>
        </div>

        <!--上拉加载提示 start-->
        <div class="pullup-tip" v-if="pullUpConfig">
          <slot name="pullupTip">
            <slot name="pullUpLoading">
              <div class="scroll-loading-wrap" v-show="pullupData.loading">
                <div class="scroll-loading">
                  <svg viewBox="25 25 50 50" class="loading-circular">
                    <circle cx="50" cy="50" r="20" fill="none"></circle>
                  </svg>
                </div>
                <span class="scroll-loading-text"><!--加载中...-->{{loadingText}}</span>
              </div>
            </slot>

            <div class="pullup-load-error" v-show="pullUpLoadError" @click="rePullupLoad" v-html="pullUpFetchDataErrorText"><!--请求失败，点击重新加载--></div>
            <div class="pullup-no-more-data" v-show="pullUpLoadNoMoreData" v-html="pullUpFetchDataNoMoreText"><!--已经到底部，没有更多内容了--></div>
          </slot>
        </div>
        <!--上拉加载提示 end-->
      </div>
    </div>
  </section>
</template>

<script>
  import index from "./vue-scroll-refresh-load.js";

  export default index;
</script>
<style lang="less">
  @import "vue-scroll-refresh-load";
</style>
