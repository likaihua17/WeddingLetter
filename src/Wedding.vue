<template>
  <div class="wedding">
    <Editor/>
  </div>
</template>

<script>
  import Editor from "./components/Editor.vue"
  export default {
    props: [],
    components: {
      Editor
      //video
    },
    name: 'Wedding'
  }
</script>

<style lang="less">
@import './assets/base.less';
html,
body{
  height: 100%;
  margin: 0;
  padding: 0;
}
.wedding{
  position: relative;
  max-width: 568px!important;
  height: 100%;
  min-height: 100%;
  margin: 0 auto;
  background: #2B2B48;
  perspective: 500px;
  padding: 0; /* 确保内边距为0 */
  margin-top: 0; /* 确保顶部外边距为0 */
}
</style>
