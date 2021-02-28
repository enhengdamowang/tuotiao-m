<template>
  <div class="article-list">
    <van-pull-refresh
      v-model="isRefreshLoading"
      @refresh="onRefresh"
      :success-text="refreshSuccessText"
      :success-duration="1500"
    >
      <van-list
        v-model="loading"
        :finished="finished"
        finished-text="没有更多了"
        :error.sync="error"
        error-text="请求失败，点击重新加载"
        @load="onLoad"
      >
        <!-- <van-cell
          v-for="(article, index) in list"
          :key="index"
          :title="article.title"
        ></van-cell> -->

        <article-item
          v-for="(article, index) in list"
          :key="index"
          :article="article"
        ></article-item>
      </van-list>
    </van-pull-refresh>
  </div>
</template>
<script>
import { getArticles } from '@/api/article'
import ArticleItem from '@/components/article-item'
export default {
  name: 'ArticleList',
  components: {
    ArticleItem
  },
  props: {
    channel: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      list: [], // 存储列表数据的数组
      loading: false, // 控制加载中 loading 状态，默认不 loading
      finished: false, // 数据是否加载完成
      timestamp: null, // 请求获取下一页数据的时间戳
      error: false, // 控制列表失败的提示状态
      isRefreshLoading: false, // 控制下拉刷新的 loading 状态
      refreshSuccessText: '刷新成功' // 下拉刷新成功提示文本
    }
  },
  methods: {
    async onLoad () {
      try {
        // 1. 请求获取数据
        const { data } = await getArticles({
          channel_id: this.channel.id, // 频道 ID
          timestamp: this.timestamp || Date.now(),
          // 请求数据的页码，请求第 1 页数据，用当前最新时间戳
          // 用于请求之后数据的时间戳会在当前请求的返回值中给出
          with_top: 1 // 是否包含置顶
        })
        // console.log(data)
        // 2. 把请求结果数据放到 list 数组中
        const { results } = data.data
        this.list.push(...results)
        // 3. 本次数据加载完毕后要把 loading 设置为 false，loading 关闭后才能触发下一次的加载更多
        this.loading = false
        // 4. 数据加载完毕后，要把 finished 设置为 true，不再触发加载更多了
        if (results.length) {
          // 更新获取下一页数据的时间戳
          this.timestamp = data.data.pre_timestamp
        } else {
          // 没有数据了，设置加载状态结束，不再触发上拉加载更多了
          this.finished = true
        }
      } catch (err) {
        console.log(err)
        this.error = true // 开启错误提示
        this.loading = false // 关闭 loading 效果
      }
    },
    async onRefresh () {
      try {
        // 请求获取数据
        const { data } = await getArticles({
          channel_id: this.channel.id,
          timestamp: Date.now(),
          with_top: 1
        })
        /* if (Math.random() > 0.5) {
          JSON.parse('xxx')
        } */
        // 将数据追加到列表的顶部
        const { results } = data.data
        this.list.unshift(...results)
        // 关闭下拉刷新的 loading 状态
        this.isRefreshLoading = false
        // 更新下拉刷新成功提示的文本
        this.refreshSuccessText = `刷新成功，更新了${results.length}条数据`
      } catch (err) {
        // 关闭下拉刷新的 loading 状态
        this.isRefreshLoading = false
        // 更新下拉刷新成功提示的文本
        this.refreshSuccessText = '刷新失败'
      }
    }
  }
}
</script>
<style lang="less" scoped>
.article-list {
  height: 79vh;
  overflow-y: auto;
}

</style>
