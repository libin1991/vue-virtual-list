<!-- ListView -->
<template>
  <div class="list-view" ref="list" @scroll="handleScroll">
    <div class="list-view__phantom" :style="{height: contentHeight + 'px'}"></div>
    <div class="list-view__content" ref="content">
      <div
      class="list-view__item"
      :style="{height: itemSizeGetter(item) + 'px'}"
      v-for="(item,index) in visibleData"
      :key="index"
      >
      {{ item.value }}
    </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      required: true
    },
    itemSizeGetter: {
      type: Function
    }
  },
  data () {
    return {
      visibleData: []
    };
  },

  computed: {
    contentHeight() {
      const { data, itemSizeGetter } = this;
      let total = 0;
      for (let i = 0, j = data.length; i < j; i++) {
        total += itemSizeGetter.call(null, data[i], i);
      }
      return total;
    }
  },

  methods: {
    getItemSizeAndOffset(index) {
      const { data, itemSizeGetter } = this;
      let total = 0;
      for(let i = 0, j = Math.min(index, data.length -1 ); i <= j; i++) {
        const size = itemSizeGetter.call(null, data[i]);
        if(i === j) {
          return {
            offset: total,
            size
          }
        }
        total +=size;
      }
      return {
        offset: 0,
        size: 0
      }
    },
    findNearestItemIndex(scrollTop) {
      const { data, itemSizeGetter } = this;
      let total = 0;
      for(let i = 0, j = data.length; i < j; i++) {
        const size = itemSizeGetter.call(null, data[i]);
        total += size;
        if(total >= scrollTop || i === j - 1) {
          return i
        }
      }
      return 0;
    },
    updateVisibleData(scrollTop) {
      scrollTop = scrollTop || 0;
      const start = this.findNearestItemIndex(scrollTop);
      const end = this.findNearestItemIndex(scrollTop + this.$el.clientHeight);
      console.log(start, end)
      this.visibleData = this.data.slice(start, Math.min(end + 1, this.data.length));
      this.$refs.content.style.webkitTransform = `translate3d(0, ${this.getItemSizeAndOffset(start).offset}px, 0)`;
    },
    handleScroll() {
      const scrollTop = this.$el.scrollTop;
      this.updateVisibleData(scrollTop)
    }
  },

  mounted() {
    this.updateVisibleData()
  }
}

</script>
<style lang='less' scoped>
.list-view {
  width: 400px;
  height: 400px;
  overflow: auto;
  position: relative;
  color: #333;
  border: 1px solid #aaa;
}

.list-view__phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.list-view__content {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
}

.list-view__item {
  padding: 5px;
  color: #666;
  line-height: 30px;
  box-sizing: border-box;
}
</style>