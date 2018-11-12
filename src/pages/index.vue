<!--  -->
<template>
  <list-view :data="data" :item-size-getter="itemSizeGetter" ></list-view>
</template>

<script>
import ListView from '@/components/ListView'
export default {
  data () {
    const data = [];
    for (let i = 0; i < 10000; i++) {
      data.push({ value: i });
    }
    return {
      data
    };
  },

  components: {
    ListView
  },

  methods: {
    itemSizeGetter(item) {
    	return 30 + item.value % 10;
    }
  }
}

</script>
<style lang='less' scoped>
</style>