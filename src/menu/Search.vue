<template lang="pug">
.search
  input(v-model="searchValue")
</template>

<script>
export default {
  props: ['value', 'search'],
  data() {
    return {
      searchValue: this.value
    }
  },
  watch: {
    searchValue() {
      this.$emit('search', this.searchValue);
    }
  }
}
</script>


<style lang="sass" scoped>
.search
  input    
    color: white
    padding: 1px 8px
    border: 1px solid white
    border-radius: 10px
    font-size: 16px
    font-family: serif
    width: 100%
    box-sizing: border-box
    background: transparent
</style>
