<template>
  <div class="output">
  <div class="list" v-for="(group, key) in selected" :key="key">
    <p class="header-list" v-show="group[0]">{{ key }}</p>
    <ul >
      <li class="item-list" v-for="item in group" :key="item">{{ item }}</li>
    </ul>
  </div>
  </div>
</template>

<script>
export default {
  name: 'SelectedValues',
  props: [
    'selected',
  ],
  data() {
    return {
      types: [],
    }
  },
  // beforeUpdate() {
  //   this.types = Object.keys(this.selected);
  // }
}
</script>

<style lang='sass'>
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@300;400&display=swap')

* 
  margin: 0
  padding: 0
  box-sizing: border-box
  font-family: 'Open Sans', sans-serif
</style>
