<template>
  <div id="app">
    <div class="flex-row flex-center">
      <div style="width: 1024px; margin-top: 40px;">
        <h2>使用说明：</h2>
        <ol>
          <li>支持从<strong>公众号回采</strong>、<strong>文章搜索</strong>和<strong>数据自助工具</strong>下载的数据文件</li>
          <li>支持同时计算相同时间周期内多个公众号的新榜指数，合并在一个文件即可</li>
          <li>文件中除第一张表以外的其他表<strong class="error">必须删除</strong>，第一张表的表名和字段<strong class="error">不要修改</strong></li>
          <li>点击<strong>选择文件</strong>选中已删除多余表的数据文件，点击<strong>解析文件</strong></li>
          <li>解析完成后，确认计算的时间周期是否准确，可以手动修改，然后点击<strong>计算指数</strong></li>
        </ol>
        <hr>
        <div style="margin-top: 40px;">
          <FileInput @parse="handleParse"/>
        </div>
        <div>
          <LogArea :logs="logs"/>
        </div>
        <div class="flex-row flex-between"> 
          <div>
            <TextInput label="开始日期" :value="startDate" v-on:change="changeStartDate"/>
            <TextInput label="结束日期" :value="endDate" v-on:change="changeEndDate"/>
          </div>
          <button :disabled="disableCalculate" @click="handleCalculate">计算指数</button>
        </div>
        <div style="margin-top: 60px;">
          <table :class="{hidden: hideResult}">
            <tr>
              <th>账号</th>
              <th>名称</th>
              <th>发布次数</th>
              <th>发布篇数</th>
              <th>总阅读数</th>
              <th>最高阅读数</th>
              <th>平均阅读数</th>
              <th>头条阅读数</th>
              <th>总点赞数</th>
              <th>新榜指数</th>
            </tr>
            <tr v-for="gh in statistics" :key="gh.account">
              <td>{{gh.account}}</td>
              <td>{{gh.name}}</td>
              <td>{{gh.publishCount}}</td>
              <td>{{gh.articleCount}}</td>
              <td>{{gh.readSum}}</td>
              <td>{{gh.readMax}}</td>
              <td>{{gh.readAvg}}</td>
              <td>{{gh.readSumOfHeadline}}</td>
              <td>{{gh.likeSum}}</td>
              <td>{{gh.nri}}</td>
            </tr>
          </table>
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
  components: {
    FileInput,
    LogArea,
    TextInput
  },
  data () {
    return {
      logs: [],
      startDate: '',
      endDate: '',
      disableCalculate: true,
      hideResult: true,
      statistics: null
    }
  },
  methods: {
    pushLog (text, isError = false) {
      const now = new Date().toTimeString().split(' ')[0];
      this.logs.push({ now: now, text: text, isError: isError });
    },
    handleParse (file) {
      this.hideResult = true;
      this.pushLog('开始解析...');
      const reader = new FileReader();
      reader.onload = function(e) {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, {type: 'array'});
        const checkFlag = this.checkSheet(workbook);

        if (checkFlag) {
          this.extractSheet(workbook);
        } else {
          this.pushLog('请删除多余的“字段说明”和“关于新榜data”表，刷新本页面后再次运行', true);
        }
      }.bind(this);
      reader.readAsArrayBuffer(file);
    },
    checkSheet (wb) {
      return wb.Sheets['关于新榜data'] ? false : true;
    },
    extractSheet (wb) {
      const firstSheetName = wb.SheetNames[0];
      if (firstSheetName != 'list' && firstSheetName != 'sheet1') {
        this.pushLog('表名无法识别，请联系开发者', true);
        return;
      }

      const firstSheet = wb.Sheets[firstSheetName];
      const options = {raw: false};

      // 根据最后一个字段判断属于哪种数据来源，然后定义不同 header 进行解析实现字段名统一
      // AG = 新版公众号回采
      // AE = 旧版公众号回采
      // AB = 文章搜索
      // Y =  数据自助工具
      const lastColNum = /:([A-Z]+)/.exec(firstSheet['!ref'])[1];

      
      if (lastColNum == 'AG') {
        options.range =  1;
        options.header =  ['name','account','author','orderNum', 'originalFlag', 'videoCount', 'publicTime', 'title', 'url', 'summary', 'content', 'clicksCount','likeCount', 'rewardCount', 'commentCount', 'commentLikeCount', 'commentReplyCount', 'commentReplyLikeCount', 'imageUrl', 'sourceUrl', 'videoUrl', 'musicUrl', 'audioUrl', 'memo', 'updateTime'];

      } else if (lastColNum == 'AE') {
        options.range = 1;
        options.header =  ['name','account','author','orderNum', 'originalFlag', 'videoCount', 'title', 'url', 'summary', 'content', 'clicksCount','likeCount', 'rewardCount', 'commentCount', 'commentLikeCount', 'imageUrl', 'sourceUrl', 'videoUrl', 'musicUrl', 'audioUrl', 'memo', 'publicTime', 'updateTime'];

      } else if (lastColNum == 'AB') {
        options.header = ['name', 'account', 'type', 'author', 'orderNum', 'originalFlag', 'title', 'url', 'summary', 'content', 'clicksCount', 'likeCount', 'imageUrl', 'sourceUrl', 'videoUrl', 'musicUrl', 'audioUrl', 'publicTime', 'updateTime'];
        options.range = 1;
        
      } else if (lastColNum != 'Y') {
        this.pushLog('字段无法解析，请使用未经调整的原始文件', true);
        return;
      }
      
      const rows = XLSX.utils.sheet_to_json(firstSheet, options);
      const dates = rows.map(row => Date.parse(row.publicTime));
      this.startDate = new Date(dates.sort()[0]).toJSON().slice(0, 10);
      this.endDate = new Date(dates.sort()[dates.length - 1]).toJSON().slice(0, 10);
      this.statistics =  this.assembleData(rows);
      this.pushLog('数据提取成功，请确认下方的开始日期和结束日期是否准确，不准确可以手动修改');
      this.disableCalculate = false;
    },
    changeStartDate (value) {
      this.startDate = value;
    },
    changeEndDate (value) {
      this.endDate = value;
    },
    checkDate () {
      const re = /\d{4}-\d{2}-\d{2}/;
      
      if (!(re.test(this.startDate) && re.test(this.endDate))) {
        this.pushLog('输入的日期格式不准确，请按照“yyyy-mm-dd”输入，如 2019-01-01，月和日不足两位时需要用 0 补齐', true);
        return false;
      }

      const startDate = new Date(this.startDate);
      const endDate = new Date(this.endDate);

      if (startDate > endDate) {
        this.pushLog('开始日期不能大于结束日期', true);
        return false;
      }

      return true;
    },
    handleCalculate () {
      if (this.checkDate()) {
        this.pushLog('开始计算...');
        const n = (Date.parse(this.endDate) - Date.parse(this.startDate)) / 86400000 + 1;
        this.pushLog(`时间周期为 ${this.startDate} ~ ${this.endDate}，共 ${n} 天`);

        // 根据算法（https://www.newrank.cn/public/about/reference.pdf）计算每个账号的新榜指数
        this.statistics = this.statistics.map(gh => {

          // 标准化常数
          const rS = n * 800000;
          const rM = 100000;
          const rA = 100000;
          const rH = n * 100000;
          const lS = n * 80000;

          // 标准化得分
          const rSI = (Math.log(gh.readSum + 1) / Math.log(rS + 1)) * 1000;
          const rMI = (Math.log(gh.readMax + 1) / Math.log(rM + 1)) * 1000;
          const rAI = (Math.log(gh.readAvg + 1) / Math.log(rA + 1)) * 1000;
          const rHI = (Math.log(gh.readSumOfHeadline + 1) / Math.log(rH + 1)) * 1000;
          const lSI = (Math.log(gh.likeSum + 1) / Math.log(lS + 1)) * 1000;

          // 加权后相加，保留 2 位小数
          const nri = (0.75 * rSI + 0.05 * rMI + 0.1 * rAI + 0.05 * rHI + 0.05 * lSI).toFixed(2);
          return {
            account: gh.account,
            name: gh.name,
            publishCount: gh.publishCount,
            articleCount: gh.articleCount,
            readSum: gh.readSum,
            readMax: gh.readMax,
            readAvg: gh.readAvg,
            readSumOfHeadline: gh.readSumOfHeadline,
            likeSum: gh.likeSum,
            nri: nri
          }
        });
        this.pushLog('🎉 完工~');
        this.hideResult = false;
      }
    },
    assembleData (rows) {

      const accounts = [];
      const datas = [];

      // 提取账号用于下一步的数据聚合
      rows.forEach(row => {
        if (accounts.indexOf(row.account) == -1) {
          accounts.push(row.account);
        } 
      });
      this.pushLog(`检测到 ${accounts.length} 个账号`);

      // 以 account 整合数据，结构为 {account, name, articles: []}
      rows.forEach(row => {
        const index = accounts.indexOf(row.account);
        if (!datas[index]) {
          datas[index] = { account: row.account, name: row.name, articles: [] };
        }
        datas[index].articles.push({
          orderNum: row.orderNum,
          clicksCount: parseInt(row.clicksCount),
          likeCount: parseInt(row.likeCount),
          publicTime: row.publicTime
        });
      });

      // 分别统计每个账号的发布次数、发布篇数、总阅读数、最高阅读数、平均阅读数、头条阅读数和总点赞数
      const statistics = datas.map(data => {

        // 发布次数 = 唯一发布时间的个数（每次发布多篇，发布时间是相同的）
        const publicTimes = [];
        data.articles.forEach(article => {
          if (publicTimes.indexOf(article.publicTime) == -1) {
            publicTimes.push(article.publicTime);
          }
        });
        const publishCount = publicTimes.length;
        const articleCount = data.articles.length;

        const readSum = data.articles.reduce((acc, article) => acc + article.clicksCount, 0);
        const readMax = data.articles.sort((a, b) => b.clicksCount - a.clicksCount)[0].clicksCount;
        const readAvg = Math.round(readSum / articleCount);
        const readSumOfHeadline = data.articles.filter(article => article.orderNum == 0).reduce((acc, article) => acc + article.clicksCount, 0);
        const likeSum = data.articles.reduce((acc, article) => acc + article.likeCount, 0);

        return {
          account: data.account,
          name: data.name,
          articleCount: articleCount,
          publishCount: publishCount,
          readSum: readSum,
          readMax: readMax,
          readAvg: readAvg,
          readSumOfHeadline: readSumOfHeadline,
          likeSum: likeSum
        }
      });

      return statistics;
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
  font-size: 0.8em;
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
li {
  padding: 2px 0;
}
.hidden {
  display: none;
}
table {
  font-size: 0.8em;
  text-align: center;
  width: 100%;
  border-collapse: collapse;
}
hr {
  display: block;
  height: 1px;
  border: 0;
  border-top: 1px solid #dddddd;
  margin: 1em 0;
  padding: 0; 
}
th, td {
  padding: 2px 0;
  border: 1px solid #dddddd;
}
</style>