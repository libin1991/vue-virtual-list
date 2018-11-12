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
  name: 'ListView',
  props: {
    data: {
      type: Array,
      required: true
    },
    estimatedItemSize: {
      type: Number,
      default: 30
    },
    itemSizeGetter: {
      type: Function
    }
  },
  data () {
    return {
      lastMeasuredIndex: -1,
      startIndex: 0,
      sizeAndOffsetCahce: [],
      visibleData: []
    };
  },

  computed: {
    contentHeight() {
			const { data, lastMeasuredIndex, estimatedItemSize } = this;
      let itemCount = data.length;
      if (lastMeasuredIndex >= 0) {
        const lastMeasuredSizeAndOffset = this.getLastMeasuredSizeAndOffset();
        return lastMeasuredSizeAndOffset.offset + lastMeasuredSizeAndOffset.size + (itemCount - 1 - lastMeasuredIndex) * estimatedItemSize;
      } else {
        return itemCount * estimatedItemSize;
      }
    }
  },

   methods: {
  	getItemSizeAndOffset(index) {
      const { lastMeasuredIndex, sizeAndOffsetCahce, data, itemSizeGetter } = this;
      if (lastMeasuredIndex >= index) {
        return sizeAndOffsetCahce[index] || { offset: 0, size: 0 };
      }

      let offset = 0;
      if (lastMeasuredIndex >= 0) {
        const lastMeasured = sizeAndOffsetCahce[lastMeasuredIndex];
        if (lastMeasured) {
          offset = lastMeasured.offset + lastMeasured.size;
        }
      }

      for (let i = lastMeasuredIndex + 1; i <= index && i < data.length; i++) {
        const item = data[i];
        const size = itemSizeGetter.call(null, item, i);
        sizeAndOffsetCahce[i] = {
          size,
          offset
        };
        offset += size;
      }
      this.lastMeasuredIndex = Math.min(index, data.length - 1);
      return sizeAndOffsetCahce[index] || { offset: 0, size: 0 };
    },
    
    getLastMeasuredSizeAndOffset() {
      return this.lastMeasuredIndex >= 0 ? this.sizeAndOffsetCahce[this.lastMeasuredIndex] : { offset: 0, size: 0 };
    },
    
    
    binarySearch(low, high, offset) {
      let index;

      while (low <= high) {
        const middle = Math.floor((low + high) / 2);
        const middleOffset = this.getItemSizeAndOffset(middle).offset;
        if (middleOffset === offset) {
          index = middle;
          break;
        } else if (middleOffset > offset) {
          high = middle - 1;
        } else {
          low = middle + 1;
        }
      }

      if (low > 0) {
        index = low - 1;
      }

      if (typeof index === 'undefined') {
        index = 0;
      }

      return index;
    },
    
    exponentialSearch(scrollTop) {
      let bound = 1;
      const data = this.data;
      const start = this.lastMeasuredIndex >= 0 ? this.lastMeasuredIndex : 0;
      while (start + bound < data.length && this.getItemSizeAndOffset(start + bound).offset < scrollTop) {
        bound = bound * 2;
      }
      return this.binarySearch(start + Math.floor(bound / 2), Math.min(start + bound, data.length), scrollTop);
    },
  
  	findNearestItemIndex(scrollTop) {
      const { data } = this;
      const lastMeasuredOffset = this.getLastMeasuredSizeAndOffset().offset;
      if (lastMeasuredOffset > scrollTop) {
        return this.binarySearch(0, this.lastMeasuredIndex, scrollTop);
      } else {
      	return this.exponentialSearch(scrollTop);
      }
    },

  	updateVisibleData(scrollTop) {
    	scrollTop = scrollTop || 0;
      const start = this.findNearestItemIndex(scrollTop);
      const end = this.findNearestItemIndex(scrollTop + this.$el.clientHeight) + 1;
      this.visibleData = this.data.slice(start, Math.min(end + 1, this.data.length));
      this.startIndex = start;
      this.$refs.content.style.webkitTransform = `translate3d(0, ${ this.getItemSizeAndOffset(start).offset }px, 0)`;
    },

    handleScroll() {
      const scrollTop = this.$el.scrollTop;
      this.updateVisibleData(scrollTop);
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