<template>
  <div class="home-container">
    <!-- 导航栏 -->
    <van-nav-bar class="app-nav-bar">
      <van-button
        slot="title"
        class="search-btn"
        icon="search"
        type="info"
        round
        size="small"
        to="/search"
      >搜索</van-button>
    </van-nav-bar>
    <!-- /导航栏 -->

    <!-- 文章频道列表 -->
    <!--
      标签页组件有一个功能，只有你第1次查看标签页的时候才会渲染里面的内容
     -->
    <van-tabs class="channel-tabs" v-model="active">
      <van-tab
        :title="channel.name"
        v-for="channel in channels"
        :key="channel.id"
      >
        <!-- 文章列表 -->
        <article-list :channel="channel" />
        <!-- /文章列表 -->
      </van-tab>
      <!-- 汉堡按钮定位把列表最后的位置给挡住了，解决办法的就是在这里添加一个占位元素 -->
      <div
        slot="nav-right"
        class="wap-nav-placeholder"
      ></div>
      <div
        slot="nav-right"
        @click="isChannelEditShow = true"
        class="wap-nav-wrap"
      >
        <van-icon name="wap-nav" />
      </div>
    </van-tabs>
    <!-- /文章频道列表 -->

    <van-popup
      v-model="isChannelEditShow"
      position="bottom"
      class="channel-edit-popup"
      closeable
      close-icon-position="top-left"
      get-container="body"
      style="height: 100%"
    >
      <!--
        模板中的 $event 表示事件参数
       -->
      <channel-edit
        :user-channels="channels"
        :active="active"
        @close="isChannelEditShow = false"
        @update-active="active = $event"
      />
    </van-popup>
  </div>
</template>

<script>
import { getUserChannels } from '@/api/user'
import ArticleList from './components/article-list'
import ChannelEdit from './components/channel-edit'
import { mapState } from 'vuex'
import { getItem } from '@/utils/storage'

export default {
  name: 'HomeIndex',
  components: {
    ArticleList,
    ChannelEdit
  },
  props: {},
  data () {
    return {
      active: 0, // 控制被激活的标签
      channels: [], // 频道列表
      isChannelEditShow: false // 控制编辑频道的显示状态
    }
  },
  computed: {
    ...mapState(['user'])
  },
  watch: {
    user () {
      this.active = 0
      this.loadChannels()
    }
  },
  created () {
    this.loadChannels()
  },
  mounted () {},
  methods: {
    async loadChannels () {
      let channels = []
      if (this.user) {
        // 已登录，请求获取线上的用户频道列表数据
        const { data } = await getUserChannels()
        channels = data.data.channels
      } else {
        // 没有登录，判断是否有本地存储的频道列表数据
        const localChannels = getItem('user-channels')

        // 如果有本地存储的频道列表则使用
        if (localChannels) {
          channels = localChannels
        } else {
          // 用户没有登录，也没有本地存储的频道，那就请求获取默认推荐的频道列表
          const { data } = await getUserChannels()
          channels = data.data.channels
        }
      }

      // 把处理好的数据放到 data 中以供模板使用
      this.channels = channels
      // 请求获取频道数据
      // const { data } = await getUserChannels()
      // this.channels = data.data.channels
    }

    // onUpdateActive (index) {
    //   this.active = index
    // }
  }
}
</script>

<style scoped lang="less">
.home-container {
  /deep/ .van-nav-bar__title {
    max-width: none;
  }
  .search-btn {
    width: 277px;
    height: 32px;
    background: #5babfb;
    border: none;
    .van-icon {
      font-size: 16px;
    }
    .van-button__text {
      font-size: 14px;
    }
  }

  .channel-tabs {
    /deep/ .van-tab {
      border-right: 1px solid #edeff3;
      border-bottom: 1px solid #edeff3;
    }
    /deep/ .van-tabs__line {
      bottom: 20px;
      width: 15px !important;
      height: 3px;
      background: #3296fa;
    }
  }

  .wap-nav-placeholder {
    width: 33px;
    flex-shrink: 0;
  }

  .wap-nav-wrap {
    position: fixed;
    right: 0;
    width: 33px;
    height: 43px;
    background-color: #fff;
    display: flex;
    justify-content: center;
    align-items: center;
    opacity: .9;
    .van-icon {
      font-size: 24px;
    }
    &:before {
      content: '';
      width: 1px;
      background: url("./line.png") no-repeat;
      background-size: contain;
      position: absolute;
      left: 0;
      top: 0;
      bottom: 0;
    }
  }
}
</style>
