<template lang="pug">
.item(
  @click="onClick($event)"
  @mouseover="showSubitems()"
  @mouseleave="timeoutHide()"
  :class="{ hasSubitems }"
) {{item.title}}
  .subitems(v-show="hasSubitems && this.visibleSubitems")
    Item(v-for="subitem in filteredSubitems"
      :key="subitem.key||subitem.title"
      :item="subitem"
      :args="args"
      :delay="delay"
      )
</template>

<script>
import hideMixin from './debounceHide'
import { EventBus } from './Search.vue';

export default {
  name: 'Item',
  mixins: [hideMixin('hideSubitems')],
  props: { item: Object, args: Object, searchKeep: Function },
  data() {
    return {
      visibleSubitems: false,
      filter: '',
    }
  },
  computed: {
    hasSubitems() {
      return this.item.subitems
    },

    filteredSubitems() {
      if(!this.item.subitems) {
        return [];
      }
      if(!this.filter) return this.item.subitems;
      const regex = new RegExp(this.filter, 'i');

      return this.item.subitems
        .filter(item => {
          return item.title.match(regex) || this.searchKeep(item, regex)
        });
    }
  },
  methods: {
    onSearch(e) {
      this.filter = e;
    },
    showSubitems() {
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
  },
  mounted() {
    EventBus.$on('search', this.onSearch);
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
