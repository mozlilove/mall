<template>
  <div class="article" v-if="article">
    <top-bar/>

    <div class="video">
      <video width="100%" controls="controls" :src="article.content" />
    </div>
    <div class="article-Info">

              <div class="user_img">
          <img v-if="article.userinfo.user_img" :src="article.userinfo.user_img">
          <img v-else src="~assets/img/default_img.jpg">
        </div>
      <div class="article-user">
        <span class="user_name">{{article.userinfo.name}}</span>
        <div class="user_subscription" @click="Subscribe" :class="{SubscriptionColor:subscribe}">
          <img v-if="!subscribe" src="~assets/img/like.svg">
          {{subscribe?'已关注':'关注'}}
          </div>
      </div>

      <div class="title">
        <div class="left">
          <span class="category">{{ article.category.title }}</span>
          <span class="name">{{ article.name }}</span>
        </div>
        <div class="right">
          <van-icon name="arrow-down" />
        </div>
      </div>

      <div class="data">
        <span class="play">2549.5w观看</span>
        <span class="dmcount">{{
          article.commentlen * Math.floor(Math.random()*10000+1)
        }}弹幕</span>
        <span class="date">{{ article.date }}</span>
      </div>

      <div class="video-toolbar">
        <span class="video-star" @click="collect" :class="{activeChose:activeColor}"><img src="~assets/img/star.svg">收藏</span>
        <span class="video-download" @click="download"><img src="~assets/img/download.svg">下载</span>
        <span class="video-share" @click="share"><img src="~assets/img/share.svg">分享</span>
      </div>
    </div>


    

    <div class="recommend">
        <detail class="detail" v-for="(item ,index) in recommend"  :key="index" :detailItem="item" />
    </div>

    <comment :article="article" @submitComment="dealComment" ref="commentRef" @replyCommentId="getReplyCommentId"></comment>
  </div>
</template>

<script>
import TopBar from "components/content/topbar/TopBar";
import Comment from "components/content/comment/Comment";
import Detail from 'views/home/Detail'

import request from "network/request";
export default {
  name: "Article",
  data() {
    return {
      article: null,
      recommend:[],
      postComment:{
        comment_content:'',
        comment_date:'',
        article_id:null,
        parent_id:null
      },
      activeColor:null,
      subscribe:null
    };
  },
  components: {
    TopBar,
    Detail,
    Comment
  },
  methods: {
    //获取信息
    getArticle() {
      request
        .get("/article/" + this.$route.params.id)
        .then(res => {
          console.log(res.data[0]);
          this.article = res.data[0];
          if(this.article){
            this.getSubscription()
          }
        })
        .catch(err => {});
    },
    getRecommend() {
        request.get(
            '/commend'
        ).then(res => {
            this.recommend = res.data
        }).then(err => {

        })
    },
    dealComment(comment){
      const date = new Date()
      let m = date.getMonth() + 1
      let d = date.getDate()
      if(m < 10){
        m = '0' + m
      }
      if(d < 10){
        d = '0' + d
      }
      let dateStr = `${m}-${d}`
      this.postComment.comment_content = comment
      this.postComment.comment_date = dateStr
      this.postComment.article_id = this.$route.params.id

      request.post(
        '/comment_post/'+ localStorage.getItem('id'),
        this.postComment
      ).then(res => {
        // console.log(res);
        this.$toast.fail("评论成功");
        //刷新，从新获取评论
        this.$refs.commentRef.content = ''
        this.$refs.commentRef.getCommentInfo()
      }).catch(err => {
        // console.log(err);
        this.$toast.fail("出了点小错误");
      })
    },
    //获取回复目标评论的id
    getReplyCommentId(id) {
      this.postComment.parent_id = id
    },
    //关注用户
    Subscribe() {
           if(localStorage.getItem('id')&&localStorage.getItem('token')){
        request.post(
          '/sub_scription/'+localStorage.getItem('id'),
          {
            sub_id:this.article.userid
          }
        ).then(res => {
          if(res.data.msg === '关注成功'){
            this.$toast.fail('关注成功')
            this.subscribe = true
          }else if( res.data.msg === '取消关注成功'){
            this.$toast.fail('取消关注成功')
            this.subscribe = false
          }
        }).catch(err => {
          console.log(err);
        })
      }
    },
    //收藏文章
    collect() {
      if(localStorage.getItem('id')&&localStorage.getItem('token')){
        request.post(
          '/collection/'+localStorage.getItem('id'),
          {
            article_id:this.$route.params.id
          }
        ).then(res => {
          if(res.data.msg === '收藏成功'){
            this.$toast.fail('收藏成功')
            this.activeColor = true
          }else if( res.data.msg === '取消收藏成功'){
            this.$toast.fail('取消收藏成功')
            this.activeColor = false
          }
        }).catch(err => {
          console.log(err);
        })
      }
    },
    download() {

    },
    share() {

    },
    //获取收藏状态
    getCollection() {
      if(localStorage.getItem('token')&& localStorage.getItem('id')){
        request.get(
          '/collection/'  + localStorage.getItem('id'),
          {
            params:{
              article_id:this.$route.params.id
            }
          }
        ).then(res => {
          this.activeColor =res.data.success
        })
      }
    },
    //获取订阅状态
    getSubscription() {
           if(localStorage.getItem('token')&& localStorage.getItem('id')){
        request.get(
          '/sub_scription/'  + localStorage.getItem('id'),
          {
            params:{
              sub_id:this.article.userid
            }
          }
        ).then(res => {
          this.subscribe =res.data.success
        })
      }
    }
  },
  watch:{
      $route() {
          this.getArticle();
          this.getRecommend();
          this.getCollection()
      }
  },
  created() {
    this.getArticle();
    this.getRecommend();
    this.getCollection()
  }
}
</script>

<style lang="less">
.article {
  background-color: #fff;
}
.video {
  width: 100%;
}
.article-Info {
  margin-top: 4vw;
  padding: 0 3.2vw;
  border-bottom: 0.133vw solid #eee;
  position: relative;
      .user_img{
      display: inline-block;
      padding-right: 2vw;
      position: absolute;
      img{
        width:8vw;
        height: 8vw;
        vertical-align: middle;
      }
      
    }
  .article-user{
    display: flex;
    justify-content: space-between;
    margin-left: 10vw;
    line-height: 10vw;
    .user_name::before{
       display: inline-block;
      content: '';
      height: 100%;
      vertical-align: middle;
    }
    .user_name{
      vertical-align: middle;
      color: #212121;
    }
    .user_subscription{
      background-color: #fb7299;
      border-radius: 1vw;
      font-size: 4vw;
      color: #fff;
      vertical-align: middle;
      padding: 0 1vw;
      img{
       width: 5vw;
       height: 5vw;
       vertical-align: text-bottom;
      }

    }
  }
  .title {
    position: relative;
    display: flex;
    justify-content: space-between;
    padding: 2vw 0;
    .left {
      display: flex;
      justify-content: space-between;
      width: 80%;
    }
    .category::before {
      display: inline-block;
      content: "";
      height: 100%;
      vertical-align: middle;
    }
    .category {
      display: inline-block;
      line-height: 2vw;
      padding: 0 1.6vw;
      margin-right: 2vw;
      border-radius: 3.2vw;
      font-size: 3vw;
      color: #fb7299;
      background-color: #eee;
    }
    .name {
      font-size: 4.26667vw;
      font-weight: 400;
      display: inline-block;
      -webkit-box-sizing: border-box;
      box-sizing: border-box;
      color: #212121;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      width: 80%;
      height: 6.4vw;
      line-height: 6.4vw;
    }
  }
  .data {
    display: flex;
    font-size: 3.2vw;
    color: #999;
    padding-bottom: 2vw;
    span{
      display: inline-block;
      margin-right: 2vw;
    }
  }
  .video-toolbar{
    font-size: 3vw;
    color: #999999;
    padding: 2vw 0;
    img{
      width: 6vw;
      height: 6vw;
      vertical-align: middle;
      margin-right: 1.2vw;
    }
    .video-star,.video-download,.video-share{
      margin-right: 3.2vw;
    }
  }
}
.recommend{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
     .detail {
        width: 45%;
        margin-top: 3.2vw;
    }
}
.activeChose{
  color: #fb7299;
}
</style>
