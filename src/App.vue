<template>
  <div id="app">
    <div class="flex-row flex-center">
      <div style="width: 720px; margin-top: 40px;">
        <h2>使用说明：</h2>
        <ol>
          <li>选择公众号回采的数据表</li>
          <li>点击<strong>解析数据</strong></li>
          <li>解析完成后，确认计算的时间周期是否准确，不准确可以手动调整，然后点击<strong>计算指数</strong></li>
          <li>指数计算完成后将显示在下方表格</li>
        </ol>
        <hr>
        <div style="margin-top: 40px;">
          <FileInput v-on:parse="handleParse" />
        </div>
        <div>
          <LogArea :logs="logs"/>
        </div>
        <div class="flex-row flex-between"> 
          <div>
            <TextInput placeholder="开始日期"/>
            <TextInput placeholder="结束日期"/>
          </div>
          <button @click="handleCalculate">计算指数</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import FileInput from './components/FileInput.vue';
import LogArea from './components/LogArea.vue';
import TextInput from './components/TextInput.vue';
import XLSX from 'xlsx';

export default {
  name: 'app',
  data () {
    return {
      logs: []
    }
  },
  components: {
    FileInput,
    LogArea,
    TextInput
  },
  methods: {
    pushLog (text, isError = false) {
      const now = new Date().toTimeString().split(' ')[0];
      this.logs.push({ now: now, text: text, isError: isError });
    },
    handleParse (file) {
      const reader = new FileReader();
      reader.onload = function(e) {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, {type: 'array'});
        const first_sheet_name = workbook.SheetNames[0];
        const sheet = {};

        if (first_sheet_name == 'Sheet1') {
          sheet.type = 1;
        } else if (first_sheet_name == 'list') {
          sheet.type = 2;
        } else {
          sheet.type = 3;
        }

        sheet.data = XLSX.utils.sheet_to_json(workbook.Sheets[first_sheet_name]);
        console.log(sheet);
      }
      reader.readAsArrayBuffer(file);
    },
    handleCalculate () {
      return;
    }
  },
  created () {
    this.pushLog('🚀 程序启动');
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
label, button, input{
  display: inline-block;
  height: 32px;
  padding: 0 10px;
  line-height: 32px;
  border-radius: 4px;
  border: 1px solid #dddddd;
  font-size: 0.6em;
  background-color: #f2f2f2;
}
.flex-row {
  display: flex;
  flex-direction: row;
}
.flex-center {
  justify-content: center;
}
.flex-between {
  justify-content: space-between;
}
strong {
  display: inline-block;
  padding: 0 6px;
  font-size: 0.9em;
  background-color: #f2f2f2;
  margin: 0 4px;
  border-radius: 2px;
}
.error {
  color: #ff6666;
}
</style>