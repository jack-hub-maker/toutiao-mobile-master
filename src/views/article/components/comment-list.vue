<template>
  <div class="comment-list">
    <van-cell title="全部评论"></van-cell>
    <van-list
      v-model="loading"
      :finished="finished"
      finished-text="没有更多了"
      @load="onLoad"
    >
      <comment-item
        v-for="(comment, index) in list"
        :key="index"
        :comment="comment"
        @reply-click="$emit('reply-click', $event)"
      />
      <!-- <van-cell
        v-for="(comment, index) in list"
        :key="index"
        :title="comment.content"
      /> -->
    </van-list>
  </div>
</template>

<script>
import { getComments } from '@/api/comment'
import CommentItem from './comment-item'

export default {
  name: 'CommentList',
  components: {
    CommentItem
  },
  props: {
    // 如果获取文章评论，则传递文章 ID
    // 如果获取评论回复，则传递评论 ID
    source: {
      type: [Number, String, Object],
      required: true
    },

    // 获取文章评论，使用字符 a
    // 获取评论回复，使用字符 c
    type: {
      type: String,
      default: 'a'
    },
    list: {
      type: Array,
      // 数组或对象的默认值必须通过函数返回
      default: () => []
      // default: function () {
      //   return []
      // }
    }
  },
  data () {
    return {
      // list: [],
      loading: false,
      finished: false,
      offset: null, // 获取下一页数据的页码
      limit: 10 // 每页大小
    }
  },
  computed: {},
  watch: {},
  created () {},
  mounted () {},
  methods: {
    async onLoad () {
      // 1. 请求获取数据
      const { data } = await getComments({
        type: this.type, // 评论类型，a-对文章(article)的评论，c-对评论(comment)的回复
        source: this.source.toString(), // 源id，文章id或评论id
        offset: this.offset, // 获取评论数据的偏移量，值为评论id，表示从此id的数据向后取，不传表示从第一页开始读取数据
        limit: this.limit // 每页大小
      })

      this.$emit('update-total-count', data.data.total_count)

      // 2. 把数据放到列表中
      const { results } = data.data
      this.list.push(...results)

      // 3. 将本次的 loading 关闭
      this.loading = false

      // 4. 判断是否还有数据
      if (results.length) {
        // 如果有，更新获取下一页数据的页码
        this.offset = data.data.last_id
      } else {
        // 如果没有，则将 finished 设置为 ture，不再触发加载更多了
        this.finished = true
      }
    }
  }
}
</script>

<style scoped lang="less"></style>
