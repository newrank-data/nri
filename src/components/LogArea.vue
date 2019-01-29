<template>
  <ul>
    <li v-for="(log, index) in logs" :key="index">[{{log.now}}]&nbsp;&nbsp;<span :class="{error: log.isError}">{{log.text}}</span></li>
  </ul>
</template>

<script>
export default {
  name: 'LogArea',
  props: ['logs'],
  watch: {
    logs () { this.$nextTick(() => { this.$el.scrollTop = this.$el.scrollHeight - 120 })}
  }
}
</script>

<style scoped>
ul {
  height: 100px;
  overflow-y: scroll;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #dddddd;
}
li {
  list-style-type: none;
  font-size: 0.8em;
}
</style>
