<template>
  <div class="shelf-item">
    <component :is="item"></component>
  </div>
</template>

<script>
  import {storeShelfMixin} from "../../utils/mixin";
  import ShelfBook from './ShelfItemBook';
  import ShelfCategory from './ShelfItemCategory';
  import ShelfAdd from './ShelfItemAdd';

  export default {
    name: "ShelfItem",
    mixins: [storeShelfMixin],
    data() {
      return {
        book: ShelfBook,
        category: ShelfCategory,
        add: ShelfAdd,
      };
    },
    props: {
      data: Object
    },
    computed: {
      item() {
        return this.data.type === 1 ? this.book : (this.data.type === 2 ? this.category : this.add);
      }
    }
  };
</script>

<style lang="scss" scoped>
  @import "../../assets/styles/global";

  .shelf-item {
    width: 100%;
    height: 100%;
    background: #346cb9;
  }
</style>
