<template>
  <div class="post-comment">
    <van-field
      v-model.trim="message"
      rows="2"
      autosize
      type="textarea"
      maxlength="50"
      placeholder="请输入留言"
      show-word-limit
    />
    <van-button
      size="mini"
      @click="onPost"
      :disabled="!message"
    >发布</van-button>
  </div>
</template>

<script>
import { addComment } from '@/api/comment'

export default {
  name: 'PostComment',
  components: {},
  props: {
    // 如果是发布文章评论，则传递文章 ID
    // 如果是发布评论回复，则传递评论 ID
    target: {
      type: [Number, String, Object],
      required: true
    },

    // 如果是发布评论回复，则也需要传递文章 ID
    articleId: {
      type: [Number, String, Object],
      default: null
    }
  },
  data () {
    return {
      message: '' // 输入框内容
    }
  },
  computed: {},
  watch: {},
  created () {},
  mounted () {},
  methods: {
    async onPost () {
      this.$toast.loading({
        message: '发布中',
        forbidClick: true // 禁止背景点击
      })
      // 找到数据接口
      // 封装请求方法
      // 请求提交数据
      const { data } = await addComment({
        target: this.target.toString(), // 评论的目标id（评论文章即为文章id，对评论进行回复则为评论id）
        content: this.message, // 评论的内容
        art_id: this.articleId ? this.articleId.toString() : null // 文章id，对评论内容发表回复时，需要传递此参数，表明所属文章id。对文章进行评论，不要传此参数。
      })

      this.$emit('post-success', data.data.new_obj)
      this.$toast.success('发布成功')
      // 发布成功，清空文本框内容
      this.message = ''
    }
  }
}
</script>

<style scoped lang="less">
.post-comment {
  padding: 14px;
  display: flex;
  align-items: center;
}
</style>
