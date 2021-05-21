<template>
  <div class="article-container">
    <!-- 导航栏 -->
    <van-nav-bar
      class="app-nav-bar"
      title="文章详情"
      left-arrow
      @click-left="$router.back()"
    />
    <!-- /导航栏 -->

    <div class="article-wrap">
      <h1 class="title">{{ article.title }}</h1>
      <van-cell center class="user-info">
        <div slot="title" class="name">{{ article.aut_name }}</div>
        <van-image
          slot="icon"
          class="avatar"
          round
          fit="cover"
          :src="article.aut_photo"
        />
        <div slot="label" class="pubdate">{{ article.pubdate | relativeTime }}</div>
        <van-button
          class="follow-btn"
          :type="article.is_followed ? 'default' : 'info'"
          :icon="article.is_followed ? '' : 'plus'"
          round
          size="small"
          :loading="isFollowLoading"
          @click="onFollow"
        >{{ article.is_followed ? '已关注' : '关注' }}</van-button>
      </van-cell>
      <div
        class="markdown-body"
        v-html="article.content"
        ref="article-content"
      ></div>
      <!-- 文章评论列表 -->
      <comment-list
        :source="articleId"
        :list="commentList"
        @update-total-count="totalCommentCount = $event"
        @reply-click="onReplyClick"
      />
      <!-- /文章评论列表 -->
    </div>

    <!-- 底部区域 -->
    <div class="article-bottom">
      <van-button
        class="comment-btn"
        type="default"
        round
        size="small"
        @click="isPostShow = true"
      >写评论</van-button>
      <van-icon
        name="comment-o"
        :info="totalCommentCount"
        color="#777"
      />
      <van-icon
        :color="article.is_collected ? 'orange' : '#777'"
        :name="article.is_collected ? 'star' : 'star-o'"
        @click="onCollect"
      />
      <van-icon
        :color="article.attitude === 1 ? 'hotpink' : '#777'"
        :name="article.attitude === 1 ? 'good-job' : 'good-job-o'"
        @click="onLike"
      />
      <van-icon name="share" color="#777777"></van-icon>
    </div>
    <!-- /底部区域 -->

    <!-- 发布评论 -->
    <van-popup
      v-model="isPostShow"
      position="bottom"
    >
      <post-comment
        :target="articleId"
        @post-success="onPostSuccess"
      />
    </van-popup>
    <!-- /发布评论 -->

    <!-- 评论回复 -->
    <van-popup
      v-model="isReplyShow"
      position="bottom"
    >
      <!-- 这里使用 v-if 的目的是让组件随着弹出层的显示进行渲染和销毁，防止加载过的组件不重新渲染导致数据不会重新加载的问题 -->
      <comment-reply
        v-if="isReplyShow"
        :comment="replyComment"
        :article-id="articleId"
        @close="isReplyShow = false"
      />
    </van-popup>
    <!-- /评论回复 -->
  </div>
</template>

<script>
import './github-markdown.css'
import {
  getArticleById,
  addCollect,
  deleteCollect,
  addLike,
  deleteLike
} from '@/api/article'
import { ImagePreview } from 'vant'
import { addFollow, deleteFollow } from '@/api/user'
import CommentList from './components/comment-list'
import PostComment from './components/post-comment'
import CommentReply from './components/comment-reply'

export default {
  name: 'ArticleIndex',
  components: {
    CommentList,
    PostComment,
    CommentReply
  },
  // 在组件中获取动态路由参数：
  //    方式一：this.$route.params.articleId
  //    方式二：props 传参，推荐
  //      this.articleId
  props: {
    articleId: {
      type: [String, Number, Object],
      required: true
    }
  },
  data () {
    return {
      article: {}, // 文章数据对象
      isFollowLoading: false, // 关注用户按钮的 loading 状态
      isCollectLoading: false, // 收藏的 loading 状态
      isPostShow: false, // 控制发布评论的显示状态
      commentList: [], // 文章评论列表
      totalCommentCount: 0, // 评论总数据量
      isReplyShow: false, // 控制回复的显示状态
      replyComment: {} // 当前回复评论对象
    }
  },
  computed: {},
  watch: {},
  created () {
    this.loadArticle()
  },
  mounted () {},
  methods: {
    async loadArticle () {
      const { data } = await getArticleById(this.articleId)
      this.article = data.data

      // 数据改变影响视图更新（DOM数据）不是立即的
      // 所以如果需要在修改数据之后马上操作被该数据影响的视图 DOM，需要把这个代码放到 $nextTick 中

      // this.$nextTick 是 Vue 提供的一个方法
      // 参考文档：
      this.$nextTick(() => {
        this.handlePerviewImage()
      })
    },

    handlePerviewImage () {
      // 1. 获取文章内容 DOM 容器
      const articleContent = this.$refs['article-content']

      // 2. 得到所有的 img 标签
      const imgs = articleContent.querySelectorAll('img')

      const imgPaths = [] // 收集所有的图片路径

      // 3. 循环 img 列表，给 img 注册点击事件
      imgs.forEach((img, index) => {
        imgPaths.push(img.src)
        img.onclick = function () {
          // 4. 在事件处理函数中调用 ImagePreview() 预览
          ImagePreview({
            images: imgPaths, // 预览图片路径列表
            startPosition: index // 起始位置
          })
        }
      })
    },

    async onFollow () {
      this.isFollowLoading = true
      if (this.article.is_followed) {
        // 已关注，取消关注
        await deleteFollow(this.article.aut_id)
        // this.article.is_followed = false
      } else {
        // 没有关注，添加关注
        await addFollow(this.article.aut_id)
        // this.article.is_followed = true
      }
      this.article.is_followed = !this.article.is_followed
      this.isFollowLoading = false
    },

    async onCollect () {
      this.$toast.loading({
        message: '操作中...',
        forbidClick: true // 禁止背景点击
      })
      if (this.article.is_collected) {
        // 已收藏，取消收藏
        await deleteCollect(this.articleId)
      } else {
        // 没有收藏，添加收藏
        await addCollect(this.articleId)
      }
      this.article.is_collected = !this.article.is_collected
      this.$toast.success(`${this.article.is_collected ? '' : '取消'}收藏成功`)
    },

    async onLike () {
      this.$toast.loading({
        message: '操作中...',
        forbidClick: true // 禁止背景点击
      })
      if (this.article.attitude === 1) {
        // 已点赞，取消点赞
        await deleteLike(this.articleId)
        this.article.attitude = -1
      } else {
        // 没有点赞，添加点赞
        await addLike(this.articleId)
        this.article.attitude = 1
      }
      this.$toast.success(`${this.article.attitude === 1 ? '' : '取消'}点赞成功`)
    },

    onPostSuccess (comment) {
      // 把发布成功的评论数据对象放到评论列表顶部
      this.commentList.unshift(comment)

      // 更新评论的总数量
      this.totalCommentCount++

      // 关闭发布评论弹出层
      this.isPostShow = false
    },

    onReplyClick (comment) {
      console.log('onReplyClick', comment)
      this.replyComment = comment

      // 展示回复内容
      this.isReplyShow = true
    }
  }
}
</script>

<style scoped lang="less">
.article-wrap {
  position: fixed;
  left: 0;
  right: 0;
  top: 46px;
  bottom: 44px;
  overflow-y: auto;
}
.title {
  font-size: 20px;
  color: #3a3a3a;
  padding: 24px 20px 18px;
  background-color: #fff;
  margin: 0;
}

.user-info {
  .avatar {
    width: 35px;
    height: 35px;
    margin-right: 8px;
  }
  .name {
    font-size: 12px;
    color: #333333;
  }
  .pubdate {
    font-size: 11px;
    color: #b4b4b4;
  }
  .follow-btn {
    width: 85px;
    height: 29px;
  }
}

ul {
  list-style: unset;
}

.markdown-body {
  padding: 14px;
  background-color: #fff;
}

.article-bottom {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  box-sizing: border-box;
  height: 44px;
  border-top: 1px solid #d8d8d8;
  background-color: #fff;
  .comment-btn {
    width: 141px;
    height: 23px;
    border: 1px solid #eeeeee;
    font-size: 15px;
    line-height: 23px;
    color: #a7a7a7;
  }
  .van-icon {
    font-size: 20px;
    .van-info {
      font-size: 11px;
      background-color: #e22829;
    }
  }
}
</style>
