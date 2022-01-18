<template>
  <div class="sidebar">
    <ul>
      <li class="logo">
        <!-- <i class="fab fa-youtube"></i> -->
        <p>YouTubeAPI</p>
      </li>
      <li v-for="list in lists" :key="list.id" class="li-hover">
        <!-- <i :class="list.i"></i> -->
        <p>{{ list.title }}</p>
      </li>
    </ul>
  </div>
</template>
<style scoped>
.sidebar{
  width: 15%;
  min-width: 225px;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  background-color:	rgb(53,57,65);
  box-shadow: 0 0 15px 10px rgba(0, 0, 0, .2);
}
ul{
  flex-direction: column;
  padding: 0;
  margin: 0;
  display: flex;
  flex-wrap: wrap;
}
li{
  list-style: none;
  border-radius: 5px;
  margin: 5px 15px;
  padding-left: 20px;
  transition: .5s;
  display: flex;
  align-items: center;
}
.li-hover:hover{
  box-shadow: 0 0 15px 1px rgba(255, 255, 255, .2);
  opacity: .7;
}
li i{
  font-size: 1.2em;
  margin-right: 20px;
}
.logo{
  margin: 20px 10px 30px;
}
.logo i{
  font-size: 1.8em;
  color: #e4504b;
  margin-top: 4px;
  margin-right: 10px;
}
.logo p{
  font-size: 1.3em;
  margin: 0;
}
</style>
<script>
export default {
  data(){
    return{
      lists:[
        { title: '検索', i: 'fas fa-search', id:1 },
        { title: '登録チャンネル', i: 'fas fa-video', id:2 },
        { title: '再生リスト', i: 'fas fa-list', id:3 }
      ]
    }
  }
}
</script>
