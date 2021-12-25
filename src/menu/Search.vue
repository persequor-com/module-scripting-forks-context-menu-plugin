<template lang="pug">
.search
  input(v-model="value", placeholder="Search")
</template>

<script>
// https://www.digitalocean.com/community/tutorials/vuejs-global-event-bus
export const EventBus = new Vue();

export const Search = {
  name: 'Search',
  data: function() {
    return {
      value: ''
    }
  },
  watch: {
    value(newValue) {
      EventBus.$emit('search', newValue);
    }
  }
}

export default Search;
</script>


<style lang="sass" scoped>
.search
  input    
    color: black
    padding: 1px 8px
    border: 1px solid white
    border-radius: 10px
    font-size: 16px
    font-family: serif
    width: 100%
    box-sizing: border-box
    background: white
</style>
