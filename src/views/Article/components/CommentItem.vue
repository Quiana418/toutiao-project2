<template>
  <van-cell>
    <van-image
      slot="icon"
      round
      width="30"
      height="30"
      style="margin-right: 10px"
      :src="item.aut_photo"
    />
    <span style="color: #466b9d" slot="title">hello</span>
    <div slot="label">
      <p style="color: #363636">{{ item.content }}</p>
      <p>
        <span style="margin-right: 10px">{{ item.pubdate | dateFormate }}</span>
        <!-- @click="$emit('replay-show', true)"子向父 传给ArticleComment -->
        <van-button
          size="mini"
          type="default"
          @click="$emit('replay-show', item)"
          >回复</van-button
        >
      </p>
    </div>
    <van-loading v-if="isLoading" />
    <van-icon
      v-else
      slot="right-icon"
      :name="item.is_liking ? 'like' : 'like-o'"
      :color="item.is_liking ? 'red' : ''"
      @click="onClick"
    >
      {{ item.like_count > 0 ? item.like_count : "赞" }}
    </van-icon>
  </van-cell>
</template>

<script>
import { addLike, delLike } from '@/api/comment'
export default {
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  created () { },
  data () {
    return {
      // 控制是否显示加载圈
      isLoading: false
    }
  },
  methods: {
    // 给评论点赞
    async onClick () {
      // Ajax开始前
      this.isLoading = true
      if (this.item.is_liking) {
        // 取消点赞
        try {
          await delLike(this.item.com_id)
          this.item.like_count--
          this.item.is_liking = false
        } catch (err) {
          console.log(err)
        }
      } else {
        // 点赞
        try {
          await addLike(this.item.com_id)
          this.item.like_count++
          this.item.is_liking = true
        } catch (err) {
          console.log(err)
        }
      }
      // Ajax结束
      this.isLoading = false
    }

  },
  computed: {},
  watch: {},
  filters: {},
  components: {}
}
</script>

<style scoped lang='less'>
// /deep/.van-loading {
//   // font-size: 16px !important;
//   width: 16px !important;
//   height: 16px !important;
// }
</style>
