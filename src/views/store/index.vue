<template>
  <div class="store">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: "index"
  };
</script>

<style lang="scss" scoped>
  @import "../../assets/styles/global";
  .store {
    background: white;
    height: 100%;
    width: 100%;
  }
</style>
