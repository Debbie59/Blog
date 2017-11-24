<template>
  <div id="app">
    <header>
      <el-row>
        <el-col :span="24"><div class="grid-content bg-purple"><router-link to="/chat"><img src="../static/img/rion1.jpg" width="30" height="30" style="margin-top:5px "/> </router-link></div></el-col>
      </el-row>
    </header>
    <div class="content">
      <router-view></router-view>
    </div>

  </div>
</template>

<script>
  export default {
    name: 'app'
  }
</script>

<style>
  body{margin:0;padding:0;}
  #app {
    width:100%;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    min-height: 100%;

  }
  .bg-purple {
    background: #fff;
  }

  .grid-content {
    border-radius: 4px;
    height: 36px;

  }
  header{height:40px;line-height:40px;}
  .content{
    position: absolute;
    top:40px;
    left:0;
    background:linear-gradient(180deg,#6ca6c3,#b5e2ec);
    right:0;
    bottom:0;
  }

  .input-div{
    display: inline-block;
    width: 75%;
    height: 30px;
    border: 1px solid #ddd;
    border-radius: 6px;
    float: left;
    background-color: #fff;
    color: #bbb;
    margin: 6px 0 6px 4%;
    text-align: left;
    line-height: 30px;
    padding-left: 8px;
  }
  .input-btn{
    display: inline-block;
    height: 30px;
    width: 30px;
    border-radius: 30px;
    font-size: 35px;
    line-height: 30px;
    border: 1px solid #bbb;
    color: #666;
    float: right;
    margin: 6px 4% 6px 0;
  }

  #chat-foot{
    position: absolute;
    bottom: 0;
    width: 100%;
    height: 44px;
    background: #f3f3f3;
  }
  header{
    height:40px;
    line-height:40px;
    background:#fff;
  }
</style>
