<!-- ListView -->
<template>
  <div class="list-view" ref="list" @scroll="handleScroll">
    <div class="list-view__phantom" :style="{height: contentHeight}"></div>
    <div class="list-view__content" ref="content">
      <div
      class="list-view__item"
      :style="{height: itemHeight + 'px'}"
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
  name: 'ListView',
  props: {
    data: {
      type: Array,
      required: true
    },
    itemHeight: {
      type: Number,
      default: 30
    }
  },

  data () {
    return {
      visibleData: []
    };
  },

  computed: {
    contentHeight() {
      return this.data.length * this.itemHeight + 'px'
    }
  },

  methods: {
    updateVisibleData(scrollTop) {
      scrollTop = scrollTop || 0;
      const visibleCount = Math.ceil(this.$el.clientHeight / this.itemHeight);
      const start = Math.floor(scrollTop / this.itemHeight);
      const end = start + visibleCount;
      this.visibleData = this.data.slice(start, end);
      this.$refs.content.style.webkitTransform = `translate3d(0, ${ start * this.itemHeight }px, 0)`;
    },
    handleScroll() {
      const scrollTop = this.$el.scrollTop;
      this.updateVisibleData(scrollTop)
    }
  },

  mounted() {
    this.updateVisibleData();
  },
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
