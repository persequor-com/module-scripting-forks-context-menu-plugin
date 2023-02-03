<template lang="pug">
.item(
  @click="onClick($event)"
  @mouseover="showSubitems()"
  @mouseleave="timeoutHide()"
  :class="{ hasSubitems }"
) {{item.title}}
  .subitems(v-show="hasSubitems && this.visibleSubitems")
    Item(v-for="subitem in item.subitems"
      :key="subitem.title"
      :item="subitem"
      :current-item="currentSubItem"
      :args="args"
      :delay="delay"
      @setCurrentItem="currentSubItem = subitem"
      )
</template>

<script>
import hideMixin from './debounceHide'

export default {
  name: 'Item',
  mixins: [hideMixin('hideSubitems')],
  props: { item: Object, args: Object, currentItem: {type:Object, default: null} },
  data() {
    return {
      visibleSubitems: false,
      currentSubItem: null
    }
  },
  computed: {
    hasSubitems() {
      return this.item.subitems
    }
  },
  watch: {
    currentItem(val) {
      if (val && val !== this.item) {
        this.hideSubitems();
      }
    }
  },
  methods: {
    showSubitems() {
      this.$emit('setCurrentItem');
      this.visibleSubitems = true;
      this.cancelHide();
    },
    hideSubitems() {
      this.visibleSubitems = false;
    },
    onClick(e) {
      e.stopPropagation();
      
      if(this.item.onClick)
        this.item.onClick(this.args);
      this.$root.$emit('hide');
    }
  }
}
</script>


<style lang="sass" scoped>
@import '../vars.sass'
@import '../common.sass'

.item
  @extend .item
  &.hasSubitems:after
    content: '►'
    position: absolute
    opacity: 0.6
    right: 5px
    top: 5px
  .subitems
    position: absolute
    top: 0
    left: 100%
    width: $width
    .subitem
      @extend .item
</style>
