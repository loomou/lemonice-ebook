<template>
  <div class="ebook-reader">
    <div id="read"></div>
  </div>
</template>

<script>
  import Epub from 'epubjs';
  import {mapGetters} from 'vuex';

  global.ePub = Epub;

  export default {
    name: "EbookReader",
    computed: {
      ...mapGetters(['fileName'])
    },
    methods: {
      initEpub() {
        const url = 'http://192.168.0.135:8081/epub/' + this.fileName + '.epub';
        console.log(url);
      }
    },
    mounted() {
      this.$store.dispatch('setFileName', this.$route.params.fileName.split('|').join('/')).then(() => {
        this.initEpub();
      });
    }
  };
</script>

<style lang="scss" scoped>
  @import "../../assets/styles/global";
</style>
