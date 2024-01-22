<template lang="pug">
.context-menu(
  ref="menu"
  v-if="visible"
  v-bind:style="style",
  @mouseleave='timeoutHide()',
  @mouseover="cancelHide()"
  @contextmenu.prevent=""
  @wheel.stop=""
)
  Search(v-if="searchBar", v-model="filter", @search="onSearch")
  Item(v-for='item in filtered'
    :key="'menu-item-' + item.id"
    :item="item"
    :current-item="currentItem"
    :args="args"
    :delay="delay / 2"
    @setCurrentItem="currentItem = item"
  )
</template>

<script>
import hideMixin from './debounceHide'
import Item from './Item.vue';
import Search from './Search.vue';
import { fitViewport } from '../utils';

export default {
  props: { searchBar: Boolean },
  mixins: [hideMixin('hide')],
  data() {
    return {
      x: 0,
      y: 0,
      visible: false,
      args: {},
      filter: '',
      items: [],
      currentItem: null,
      lastItemId: 0,
    }
  },
  computed: {
    style() {
      return {
        top: this.y+'px', 
        left: this.x+'px'
      }
    },
    filtered() {
      if(!this.filter) return this.items;
      const regex = new RegExp(this.filter, 'i');
      return this.items.map(item => this.filterItems(item, regex)).filter(item => item !== null);
    }
  },
  methods: {
    filterItems(tree, regex) {
      // Case 1: The node matches the regex, we keep it all (including children)
      if (tree.title.match(regex)) {
		return tree;
	  }

      // Case 2: We have children that match the regex, we keep them and filter out the rest
      const _this = this;
      const subitems = !tree.subitems ? [] : tree.subitems.map(child => _this.filterItems(child, regex)).filter(child => child !== null);
      if (subitems.length > 0) {
        // Clone the current node and filter its children recursively
        return {
          ...tree,
          subitems: subitems
        };
      }
      // Default: We didn't match, neither did our children
      return null;
    },
    onSearch(e) {
      this.filter = e;
    },
    show(x, y, args = {}) {
      this.visible = true;
      this.x = x;
      this.y = y;
      this.args = args;
  
      this.cancelHide();
    },
    hide() {
      this.visible = false;
    },
    additem(title, onClick, path = []) {
      let items = this.items;
      for(let level of path) {
        let exist = items.find(i => i.title === level);

        if(!exist) {
          exist = { id: this.lastItemId++, title: level, subitems: [] };
          items.push(exist)
        }

        items = exist.subitems || (exist.subitems = []);
      }

      items.push({ id: this.lastItemId++, title, onClick });
    },
  },
  updated() {
    if(this.$refs.menu) {
      [this.x, this.y] = fitViewport([this.x, this.y], this.$refs.menu)
    } 
  },
  mounted() {
    this.$root.$on('show', this.show);
    this.$root.$on('hide', this.hide);
    this.$root.$on('additem', this.additem);
  },
  components: {
    Item,
    Search
  }
}
</script>


<style lang="sass" scoped>
@import '../vars.sass'
@import '../common.sass'

.context-menu
  left: 0
  top: 0
  position: fixed
  padding: 10px
  width: $width
  margin-top: -20px
  margin-left: -$width/2
  .search
    @extend .item
</style>