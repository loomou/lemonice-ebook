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
      prevPage() {
        if (this.rendition) {
          this.rendition.prev()
        }
      },
      nextPage() {
        if (this.rendition) {
          this.rendition.next()
        }
      },
      toggleTitleAndMenu() {

      },
      initEpub() {
        const url = 'http://192.168.0.135:8081/epub/' + this.fileName + '.epub';
        this.book = new Epub(url);
        this.rendition = this.book.renderTo('read', {
          width: innerWidth,
          height: innerHeight,
          method: 'default'
        });
        this.rendition.display();
        this.rendition.on('touchstart', event => {
          this.touchStartX = event.changedTouches[0].clientX;
          this.touchStartTime = event.timeStamp;
          console.log(event);
        });
        this.rendition.on('touchend', event => {
          const offsetX = event.changedTouches[0].clientX - this.touchStartX;
          const time = event.timeStamp - this.touchStartTime;
          console.log(offsetX, time);
          if (time < 500 && offsetX > 40) {
            this.prevPage();
          } else if (time < 500 && offsetX < -40) {
            this.nextPage();
          } else {
            this.toggleTitleAndMenu();
          }
          event.preventDefault();
          event.stopPropagation();
        });
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
