<template>
  <div class="background">
    <Sidebar />
    <Search class="search" />
    <!-- <link href="https://use.fontawesome.com/releases/v5.6.1/css/all.css" rel="stylesheet"> -->
  </div>
</template>
<style scoped>
/* @import url('https://fonts.googleapis.com/css2?family=Sawarabi+Gothic&display=swap');
*{
  font-family: 'Sawarabi Gothic', sans-serif !important;
} */
.background{
  width: 100vw;
  height: 100vh;
  background-color:	rgb(53,57,65);
  color: white;
  position: fixed;
  top: 0;
  left: 0;
  overflow: scroll;
}
</style>
<script>
import Search from '~/components/Search.vue'
import Sidebar from '~/components/Sidebar.vue'
export default {
  components:{
    Search,
    Sidebar
  }
}
</script>
