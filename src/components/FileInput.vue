<template>
  <div class="flex-row flex-between">
    <div>
      <label for="file">选择文件</label>
      <input type="file" id="file" name="file" accept=".xlsx, xls" @change="handleChange"><span :class="{hidden: disabled}">&nbsp;&nbsp;{{fileName}}&nbsp;({{fileSize}})</span>
    </div>
    <button :disabled="disabled" @click="handleClick">解析文件</button>
  </div>
</template>

<script>
export default {
  name: 'FileInput',
  data () {
    return {
      file: null,
      disabled: true
    }
  },
  computed: {
    fileName () {
      return this.file ? this.file.name : ''
    },
    fileSize () {
      return this.file ?
        (this.file.size > 1048576 ? `${(this.file.size / 1048576).toFixed(1)} MB` : `${(this.file.size / 1024).toFixed(1)} KB`) : ''
    }
  },
  methods: {
    handleChange ($event) {
      this.file = $event.target.files[0];
      this.disabled = this.file ? false : true;
    },
    handleClick () {
      this.$emit('parse', this.file);
    }
  }
}
</script>

<style scoped>
input {
  display: none;
}
span {
  font-size: 0.8em;
}
</style>
