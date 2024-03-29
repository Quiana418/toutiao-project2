<template>
  <div class="article-container">
    <!-- 导航栏 -->
    <van-nav-bar
      class="page-nav-bar"
      left-arrow
      @click-left="$router.back()"
      title="黑马头条"
    ></van-nav-bar>
    <!-- /导航栏 -->

    <div class="main-wrap">
      <!-- 加载中 -->
      <div class="loading-wrap" v-if="isLoading">
        <van-loading color="#3296fa" vertical>加载中</van-loading>
      </div>
      <!-- /加载中 -->

      <template v-else>
        <!-- 加载完成-文章详情 -->
        <!-- 如果文章数据对象不为空 表示文章加载完成 -->
        <div class="article-detail" v-if="articleList.art_id">
          <!-- 文章标题 -->
          <h1 class="article-title">{{ articleList.title }}</h1>
          <!-- /文章标题 -->

          <!-- 用户信息 -->
          <van-cell class="user-info" center :border="false">
            <van-image
              class="avatar"
              slot="icon"
              round
              fit="cover"
              :src="articleList.aut_photo"
            />
            <div slot="title" class="user-name">{{ articleList.aut_name }}</div>
            <div slot="label" class="publish-date">
              {{ articleList.pubdate | dateFormate }}
            </div>
            <!-- 关注 -->
            <!-- 组件之间的传参 v-model -->
            <!-- <FollowUser
              :value="articleList.is_followed"
              @input="articleList.is_followed = $event"
            ></FollowUser> -->
            <!-- 组件之间的传参 v-model简写 -->
            <!--  :target="articleList.aut_id" 作者用户id -->
            <FollowUser
              v-model="articleList.is_followed"
              :target="articleList.aut_id"
            ></FollowUser>
          </van-cell>
          <!-- /用户信息 -->

          <!-- 文章内容 -->
          <div
            class="article-content markdown-body"
            v-html="articleList.content"
          ></div>
          <van-divider>正文结束</van-divider>

          <!-- 评论 文章id:source="articleList.art_id"  评论类型type="a"-->
          <!-- @replay-show="isReplayShow = $event"  子向父 父亲绑定事件  控制回复评论的弹出层-->
          <ArticleComment
            :source="articleList.art_id"
            type="a"
            @set-count="count = $event"
            :commentList="commentList"
            @replay-show="
              comment = $event;
              isReplayShow = true;
            "
          ></ArticleComment>
        </div>
        <!-- /加载完成-文章详情 -->

        <!-- 加载失败：404 -->
        <div class="error-wrap" v-if="is404Error">
          <van-icon name="failure" />
          <p class="text">该资源不存在或已删除！</p>
        </div>
        <!-- /加载失败：404 -->

        <!-- 加载失败：其它未知错误（例如网络原因或服务端异常） -->
        <div class="error-wrap" v-else>
          <van-icon name="failure" />
          <p class="text">内容加载失败！</p>
          <van-button class="retry-btn">点击重试</van-button>
        </div>
        <!-- /加载失败：其它未知错误（例如网络原因或服务端异常） -->
      </template>
    </div>

    <!-- 底部区域 -->
    <div class="article-bottom" v-if="!isLoading && !!articleList.art_id">
      <van-button
        @click="addCommentShow = true"
        class="comment-btn"
        type="default"
        round
        size="small"
        >写评论</van-button
      >
      <van-icon name="comment-o" :badge="count" color="#777" />

      <!-- 收藏 组件传值sync-->
      <!-- is_collected 控制是否点击了收藏 前面这个和后面呢个等价-->
      <!-- <CollectArticle
        @update:is_collected="articleList.is_collected"
      ></CollectArticle> -->
      <CollectArticle
        :is_collected.sync="articleList.is_collected"
      ></CollectArticle>

      <!-- 点赞 -->
      <!-- 组件之间的传参 v-model -->
      <!-- <GiveLike
              :value="articleList.attitude"
              @input="articleList.attitude = $event"
            ></GiveLike> -->

      <GiveLike v-model="articleList.attitude"></GiveLike>
      <!-- <van-icon color="#777" name="good-job-o" /> -->
      <van-icon
        name="share"
        color="#777777"
        @click="showShare = true"
      ></van-icon>
    </div>
    <!-- /底部区域 -->

    <!-- 分享 -->
    <van-share-sheet
      v-model="showShare"
      title="立即分享给好友"
      :options="options"
    />
    <!-- 底部弹出层 -->
    <van-popup v-model="addCommentShow" position="bottom">
      <!-- 保证每次关闭组件销毁 打开 组件重新生成 执行mounted 数据和光标都会重新生成 -->
      <AddComment
        v-if="addCommentShow"
        :target="article_id"
        @add-comment="
          commentList.unshift($event);
          addCommentShow = false;
        "
      ></AddComment>
    </van-popup>

    <!-- 控制点击回复评论的弹出层 -->
    <!-- v-if="isReplayShow" 弹出层内部是v-show显示隐藏 会有缓存 在created里面只会加载一次  如果用v-if 那么每次都会重新渲染 这样每次都会有最新的数据 -->
    <van-popup v-model="isReplayShow" position="bottom" style="height: 100%">
      <ReplayComment
        :comment="comment"
        @close="isReplayShow = $event"
        v-if="isReplayShow"
      ></ReplayComment>
    </van-popup>
  </div>
</template>

<script>
import AddComment from './components/AddComment.vue'
import 'github-markdown-css'
import { getArticle } from '@/api/article'
import { ImagePreview } from 'vant'
import ArticleComment from './components/ArticleComment.vue'
import ReplayComment from './components/ReplayComment.vue'
export default {
  name: 'ArticleIndex',
  components: { ArticleComment, AddComment, ReplayComment },
  props: {
    article_id: {
      type: [Number, String],
      required: true
    }
  },
  data () {
    return {
      // 页面刚打开显示加载中
      isLoading: true,
      // 文章数据
      articleList: {},
      // 是否为404错误状态
      is404Error: false,
      // 分享面板
      showShare: false,
      // 控制点击关注按钮的状态  false为未关注
      value: false,
      options: [
        [
          { name: '微信', icon: 'wechat' },
          { name: '朋友圈', icon: 'wechat-moments' },
          { name: '微博', icon: 'weibo' },
          { name: 'QQ', icon: 'qq' }
        ],
        [
          { name: '复制链接', icon: 'link' },
          { name: '分享海报', icon: 'poster' },
          { name: '二维码', icon: 'qrcode' },
          { name: '小程序码', icon: 'weapp-qrcode' }
        ]
      ],
      // 评论数量
      count: null,
      // 控制弹出层显示隐藏
      addCommentShow: false,
      // 评论列表 从子组件提升到父组件 状态提升
      commentList: [],
      // 控制点击回复评论的弹出层是否弹出
      isReplayShow: false,
      // 整条评论信息
      comment: {}

    }
  },
  computed: {},
  watch: {},
  async created () {
    // 请求文章
    try {
      // 路由传参
      const res = await getArticle(this.article_id)
      console.log(res)
      this.articleList = res.data.data
    } catch (err) {
      // console.log('article', err)
      // 可选链 效果同下
      // if (err.response?.status === 404) {
      //   this.is404Error = true
      // }
      if (err.response && err.response.status === 404) {
        this.is404Error = true
      }
    }
    this.isLoading = false
    // 页面加载完毕
    // 视图更新完毕的一瞬间 就会调用nextTick
    this.$nextTick(() => {
      // 获取正文中的图片 做图片预览效果
      const arr = document.querySelectorAll('.article-content img')
      if (arr.length === 0) return
      // console.log(arr)
      // 存放图片的src值
      const arr1 = []
      arr.forEach((img, index) => {
        // console.log(img.src)
        arr1.push(img.src)
        img.onclick = function () {
          ImagePreview({
            images: arr1,
            startPosition: index,
            maxZoom: 3,
            showIndicator: true
          })
        }
      })
    })
  },
  mounted () { },
  methods: {}
}
</script>

<style scoped lang="less">
.article-container {
  .main-wrap {
    position: fixed;
    left: 0;
    right: 0;
    top: 92px;
    bottom: 88px;
    overflow-y: scroll;
    background-color: #fff;
  }
  .article-detail {
    .article-title {
      font-size: 40px;
      padding: 50px 32px;
      margin: 0;
      color: #3a3a3a;
    }

    .user-info {
      padding: 0 32px;
      .avatar {
        width: 70px;
        height: 70px;
        margin-right: 17px;
      }
      .van-cell__label {
        margin-top: 0;
      }
      .user-name {
        font-size: 24px;
        color: #3a3a3a;
      }
      .publish-date {
        font-size: 23px;
        color: #b7b7b7;
      }
      .follow-btn {
        width: 170px;
        height: 58px;
      }
    }

    .article-content {
      padding: 55px 32px;
      /deep/ p {
        text-align: justify;
      }
    }
  }

  .loading-wrap {
    padding: 200px 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
  }

  .error-wrap {
    padding: 200px 32px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    .van-icon {
      font-size: 122px;
      color: #b4b4b4;
    }
    .text {
      font-size: 30px;
      color: #666666;
      margin: 33px 0 46px;
    }
    .retry-btn {
      width: 280px;
      height: 70px;
      line-height: 70px;
      border: 1px solid #c3c3c3;
      font-size: 30px;
      color: #666666;
    }
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
    height: 88px;
    border-top: 1px solid #d8d8d8;
    background-color: #fff;
    .comment-btn {
      width: 282px;
      height: 46px;
      border: 2px solid #eeeeee;
      font-size: 30px;
      line-height: 46px;
      color: #a7a7a7;
    }
    .van-icon {
      font-size: 40px;
      .van-info {
        font-size: 16px;
        background-color: #e22829;
      }
    }
  }
}
</style>
