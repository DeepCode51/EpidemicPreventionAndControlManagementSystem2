<template>
  <el-container style="height: 100%">
    <el-aside width="auto" class="sidebar-container">
      <Aside></Aside>
    </el-aside>
    <el-container>
      <el-header>
        <Header></Header>
      </el-header>
      <Tag></Tag>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
import Aside from "../components/Aside";
import Header from "../components/Header";
import Tag from "../components/Tag";
export default {
  name: 'Home',
  components: {
    Header,
    Aside,
    Tag
  }
}
</script>

<style lang="scss">
  label{
    font-weight: 700 !important;
    color: #606266 !important;
  }
  .main-class{
    padding: 10px 15px 0 15px !important;
  }
  .el-main{
    padding: 0 !important;
  }
  .el-container {
    padding: 0;
    margin: 0;
    height: 100%;
  }
  .sidebar-container {
    background-color: #304156;
    -webkit-box-shadow: 2px 0 6px rgba(0,21,41,.35);
    box-shadow: 2px 0 6px rgba(0,21,41,.35);
    z-index: 1;
    height: 100%;
    top: 0;
    bottom: 0;
    left: 0;
  }
  .el-header{
    padding: 0 !important;
    height: 50px !important;
  }

</style>
