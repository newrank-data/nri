<template>
  <div>
    <span>{{label}}&nbsp;&nbsp;</span>
    <input type="text" :value="value" @change="handleChange"/>
  </div>
</template>

<script>
export default {
  name: 'TextInput',
  props: ['label', 'value'],
  methods: {
    handleChange ($event) {
      this.$emit('change', $event.target.value);
    }
  }
}
</script>

<style scoped>
  div {
    display: inline-block;
    margin-right: 12px;
  }
  span {
    font-size: 0.8em;
  }
</style>

