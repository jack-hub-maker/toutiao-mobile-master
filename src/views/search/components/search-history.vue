<template>
  <div class="search-history">
    <van-cell
      title="搜索历史"
    >
      <div v-if="isDeleteShow">
        <!--
          prop 数据如果是引用类型（数组、对象）可以修改，
          注意这个修改指的是：user.name = 'Jack'、arr.push(123)、arr.splice(0, 1)

          但是任何 prop 数据都不能重新赋值：xxx = xxx

          如果你想要让 prop 数据 = 新的数据：让父组件自己修改
         -->
        <span
          @click="$emit('update-histories', [])"
        >全部删除</span>
        &nbsp;&nbsp;
        <span @click="isDeleteShow = false">完成</span>
      </div>
      <van-icon
        v-else
        name="delete"
        @click="isDeleteShow = true"
      />
    </van-cell>
    <van-cell
      :title="history"
      v-for="(history, index) in searchHistories"
      :key="index"
      @click="onDelete(history, index)"
    >
      <van-icon
        v-show="isDeleteShow"
        name="close"
      />
    </van-cell>
  </div>
</template>

<script>
export default {
  name: 'SearchHistory',
  components: {},
  props: {
    searchHistories: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      isDeleteShow: false // 删除的显示状态
    }
  },
  computed: {},
  watch: {},
  created () {},
  mounted () {},
  methods: {
    onDelete (history, index) {
      // 如果是删除状态，则执行删除操作
      if (this.isDeleteShow) {
        this.searchHistories.splice(index, 1)
        // 持久化处理
        // 1. 修改本地存储的数据
        // 2. 请求接口删除线上的数据
        // 无论是否登录都把数据持久化到本地
        // 如果已登录，则删除线上的历史数据
        //    没有删除单个历史记录的接口
        //    只有删除所有历史记录的接口
        // setItem('search-histories', this.searchHistories)
        return
      }

      // 非删除状态，展示搜索结果
      this.$emit('search', history)
    }
  }
}
</script>

<style scoped lang="less"></style>
