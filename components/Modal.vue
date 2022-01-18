<template>
  <div class="modal">
    <div class="background" @click="closeModal"></div>
     <div class="container" :class="{expand: isExpand}">
    <div class="icon-circle" @click="expandModal">
      <!-- <i class="fas fa-arrows-alt-h"></i> -->
    </div>
    <div class="img-box">
      <img :src="modalVideo.snippet.thumbnails.high.url" alt="">
    </div>
    <div class="text-container">
      <p class="title">{{ modalVideo.snippet.title }}</p>
      <p>{{ modalVideo.snippet.description }}</p>
      <div class="video-info">
        <p>チャンネル名：{{ modalVideo.snippet.channelTitle }}</p>
        <p>動画時間：{{ modalVideo.contentDetails.duration }}</p>
        <p>再生回数：{{ modalVideo.statistics.viewCount }}回</p>
        <p>高評価数：{{ modalVideo.statistics.likeCount }}個</p>
        <p>コメント数：{{ modalVideo.statistics.commentCount }}個</p>
        <p>投稿日：{{ modalVideo.snippet.modifiedPublishedAt }}</p>
      </div>
    </div>
    <div class="buttons">
      <button @click="openLink">この動画を視聴</button>
      <button>このチャンネルを登録</button>
      <button>再生リストに追加</button>
      <button @click="closeModal">閉じる</button>
    </div>
  </div>
  </div>
</template>
<style scoped>
.modal{
  width: 85%;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 15%;
  z-index: 100;
}
.background{
    width: 100%;
    height: 100%;
    background: red;
    background-color: rgba(255,255,255, 0.3);
    margin: 0;
}
.container{
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgb(53, 57, 65);
  width: 70%;
  height: 70%;
  /* pointer-events:none; */
  display: flex;
  align-items: center;
  flex-direction: column;
  z-index: 200;
  transition: .5s;
}
.expand{
  width: 95%;
  height: 95%;
}
.icon-circle{
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.5em;
  right: 5%;
  top: 5%;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: rgba(53, 57, 65, 1);
  transition: .5s;
  cursor: pointer;
    box-shadow: 0 0 15px 10px rgba(255, 255, 255, .05);
}
.icon-circle:hover{
  box-shadow: 0 0 15px 10px rgba(255, 255, 255, .3);
}
.icon-circle:hover i{
  animation: expand ease .5s;
}
.img-box{
  position: relative;
  width: 60%;
  height: 0;
  padding-top: calc(60% * 0.5625);
  margin-top: 5%;
}
img{
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  object-fit: cover;
}
.text-container{
    display: flex;
    flex-direction: column;
    align-items: center;
}
.text-container p{
    margin: 10px;
}
.title{
    font-size: 1.4em;
    font-weight: bold;
}
.video-info{
    display: flex;
    font-size: 0.8em;
    opacity: 0.7;
}
.buttons{
  width: 80%;
  margin:30px auto;
  display: flex;
  justify-content: center;
  align-items: center;
}
.buttons button{
  margin: 0 5px;
  width: 18%;
  padding: 8px 0;
  border-radius: 8px;
  background-color: rgb(83, 87, 95);
  color: white;
  border: 1px solid rgba(153, 157, 165, 0.5);
  cursor: pointer;
  transition: .3s;
}
.buttons button:hover{
  background-color: rgb(103, 107, 115);
}
@keyframes expand{
  0%{
    transform: scaleX(1);
  }
  50%{
    transform: scaleX(1.5);
  }
  100%{
    transform: scaleX(1);
  }
}

</style>
<script>
export default {
  props :{
    modalVideo:{
      type: Object,
      default: () => ({
        "kind": "",
        "etag": "",
        "id": "",
        "statistics": {
            "viewCount": "",
            "likeCount": "",
            "favoriteCount": "",
            "commentCount": ""
        },
        "snippet": {
            "publishedAt": "",
            "modifiedPublishedAt": "",
            "channelId": "",
            "title": "",
            "description": "",
            "thumbnails": {
                "default": {
                    "url": "",
                    "width": "",
                    "height": ""
                },
                "medium": {
                    "url": "",
                    "width": "",
                    "height": ""
                },
                "high": {
                    "url": "",
                    "width": "",
                    "height": ""
                }
            },
            "channelTitle": "",
            "liveBroadcastContent": "",
            "publishTime": "",
            "omittedTitle": ""
        },
        "videoId": "",
        "contentDetails": {
            "duration": "",
            "dimension": "",
            "definition": "",
            "caption": "",
            "licensedContent": "",
            "contentRating": {},
            "projection": ""
        }
    })
    }
  },
  data(){
    return{
      isExpand: false
    }
  },
  methods:{
    openLink() {
      console.log(1)
      window.open('https://www.youtube.com/watch?v=' + this.modalVideo.id, '_blank')
    },
    closeModal() {
        this.$emit('closeModal')
        this.isExpand = false
    },
    expandModal() {
      this.isExpand = !this.isExpand
    }
  },
  watch:{
      modalVideo(){
          this.modalVideo.snippet.modifiedPublishedAt = this.modalVideo.snippet.publishedAt.split(/T|Z/).join("-").slice(0,-1)
      }
  }
}
</script>
