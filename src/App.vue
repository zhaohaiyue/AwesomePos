<template>
  <div id="app">
    <leftNav></leftNav>
    <div class="main">
      <router-view/>
    </div>
  </div>
</template>

<script>
import leftNav from '@/components/common/leftNav'
export default {
  name: 'app',
  components:{
    leftNav
  }
}
</script>

<style>
.main{
  float: left;
  width: 95%;
  background-color: #f9fafc;
  height: 100%;
  overflow: hidden;
}
</style>
