<template>
  <div class="ebook-reader">
    <div id="read"></div>
  </div>
</template>

<script>
  import Epub from 'epubjs';
  import {ebookMixin} from "../../utils/mixin";
  import {
    getFontFamily,
    getFontSize,
    getTheme,
    saveFontFamily,
    saveFontSize,
    saveTheme
  } from "../../utils/localStorage";

  global.ePub = Epub;

  export default {
    name: "EbookReader",
    mixins: [ebookMixin],
    methods: {
      prevPage() {
        if (this.rendition) {
          this.rendition.prev();
          this.hideTitleAndMenu();
        }
      },
      nextPage() {
        if (this.rendition) {
          this.rendition.next();
          this.hideTitleAndMenu();
        }
      },
      toggleTitleAndMenu() {
        // this.$store.dispatch('setMenuVisible', !this.menuVisible);
        if (this.menuVisible) {
          this.setSettingVisible(-1);
          this.setFontFamilyVisible(false);
        }
        this.setMenuVisible(!this.menuVisible);
      },
      hideTitleAndMenu() {
        // this.$store.dispatch('setMenuVisible', false);
        this.setMenuVisible(false);
        this.setSettingVisible(-1);
        this.setFontFamilyVisible(false);
      },
      initFontSize() {
        let fontSize = getFontSize(this.fileName);
        if (!fontSize) {
          saveFontSize(this.fileName, this.defaultFontSize);
        } else {
          this.rendition.themes.fontSize(fontSize);
          this.setDefaultFontSize(fontSize);
        }
      },
      initFontFamily() {
        let font = getFontFamily(this.fileName);
        if (!font) {
          saveFontFamily(this.fileName, this.defaultFontFamily);
        } else {
          this.rendition.themes.font(font);
          this.setDefaultFontFamily(font);
        }
      },
      initTheme() {
        let defaultTheme = getTheme(this.fileName);
        if (!defaultTheme) {
          defaultTheme = this.themeList[0].name;
          saveTheme(this.fileName, defaultTheme);
        }
        this.setDefaultTheme(defaultTheme);
        this.themeList.forEach(theme => {
          this.rendition.themes.register(theme.name, theme.style);
        });
        this.rendition.themes.select(this.defaultTheme);
      },
      initRendition() {
        this.rendition = this.book.renderTo('read', {
          width: innerWidth,
          height: innerHeight,
          method: 'default'
        });
        this.rendition.display().then(() => {
          this.initFontSize();
          this.initFontFamily();
          this.initTheme();
          this.initGlobalStyle();
        });
      },
      initGesture() {
        this.rendition.on('touchstart', event => {
          this.touchStartX = event.changedTouches[0].clientX;
          this.touchStartTime = event.timeStamp;
        });
        this.rendition.on('touchend', event => {
          const offsetX = event.changedTouches[0].clientX - this.touchStartX;
          const time = event.timeStamp - this.touchStartTime;
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
      },
      renditionHooksContent() {
        this.rendition.hooks.content.register(contents => {
          Promise.all([
            contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/daysOne.css`),
            contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/cabin.css`),
            contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/montserrat.css`),
            contents.addStylesheet(`${process.env.VUE_APP_RES_URL}/fonts/tangerine.css`),
          ]).then(() => {
          });
        });
      },
      initEpub() {
        const url = process.env.VUE_APP_RES_URL + '/epub/' + this.fileName + '.epub';
        this.book = new Epub(url);
        this.setCurrentBook(this.book);
        this.initRendition();
        this.initGesture();
        this.renditionHooksContent();
        this.book.ready.then(() => {
          return this.book.locations.generate(750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16));
        }).then(() => {
          this.setBookAvailable(true);
        });
      }
    },
    mounted() {
      this.setFileName(this.$route.params.fileName.split('|').join('/')).then(() => {
        this.initEpub();
      });
    }
  };
</script>

<style lang="scss" scoped>
  @import "../../assets/styles/global";
</style>
