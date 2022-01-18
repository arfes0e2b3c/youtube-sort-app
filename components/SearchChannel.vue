<template>
  <div class="modal">
    <div class="background" @click="closeSearchChannel"></div>
    <div class="container">
      <h2>チャンネルを指定する</h2>
      <div class="search-channel">
        <input type="text" v-model="q">
        <button @click="fetchChannel">検索する</button>
      </div>
        <ul class="search-results">
          <li v-for="channel in channels" :key="channel.id.channelId" @click="selectChannel(channel)">
            <img :src="channel.snippet.thumbnails.high.url" alt="">
            <p>{{ channel.snippet.title}}</p>
          </li>
        </ul>
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
  height: 80%;
  /* pointer-events:none; */
  display: flex;
  align-items: center;
  flex-direction: column;
  z-index: 200;
  transition: .5s;
}
.search-channel{
  width: 100%;
  display: flex;
  align-items: center;
  flex-direction: column;
}
input{
    width: 50%;
    height: 30px;
    color: white;
    background: none;
    outline: none;
    border: none;
    border-bottom: 1px solid white;
    text-align: center;
    margin: 0 auto 15px;
    font-size: 1.5em;
    transition: .5s;
}
input:focus{
    width: 40%;
}
.search-channel button{
  margin: 0 5px;
  width: 15%;
  padding: 8px 0;
  border-radius: 8px;
  background-color: rgb(83, 87, 95);
  color: white;
  border: 1px solid rgba(153, 157, 165, 0.5);
  cursor: pointer;
  transition: .3s;
}
.search-channel button{
  background-color: rgb(103, 107, 115);
}
ul{
  width: 100%;
  padding: 0;
  overflow: scroll;
}
li{
  display: flex;
  justify-content: flex-start;
  align-items: center;
  width: 60%;
  margin: 10px auto;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  transition: .5s;
  box-shadow: 0 0 15px 10px rgba(0, 0, 0, .2)
}
li:hover{
  background-color:rgb(103,107,115);
}
li img{
  width: 50px;
  height: 50px;
  border-radius: 50%;
  margin: 0 50px 0 30px;
}
li p{
  margin: 0;
}
</style>
<script>
export default {
  methods:{
    async fetchChannel({ $axios }) {
      const url = "https://www.googleapis.com/youtube/v3/"
      const that = this
      const key = 'AIzaSyDQJVohnAdT4O7QSlQ0PjOy5A0Ra3oYFho'
      for(let i in that.selectedParams) {
          if(!that.selectedParams[i]) {
              delete that.selectedParams[i]
          }
      }
      await this.$axios.$get
        (url + 'search', {
          params: {
            key: key,
            part: 'snippet',
            q: this.q,
            maxResults: 50,
            order: 'relevance',
            type: 'channel'
          }
        }).catch(function(error) {
          console.log('ERROR!')
        }).then(function(response) {
          that.channels = response.items
          console.log(response.items)
        })
  },
  closeSearchChannel() {
        this.$emit('closeSearchChannel')
        this.isExpand = false
    },
  selectChannel(data) {
    this.closeSearchChannel()
    this.$emit('setChannelId', data)
  }
  },
  data() {
    return{
      q: '',
      channels: []
    }
  }
}
</script>
