<template>
  <div id="wrapper">
    <main v-loading='loading'>
      <div class="titles">
        <div class="title">
          <el-table :data="tableData" style="width: 100%">
            <el-table-column type="index" :index="index" label="#" width="50">
            </el-table-column>
            <el-table-column prop="title" label="分裂标题">
            </el-table-column>
            <el-table-column prop="price" label="价格" width="80">
            </el-table-column>
            <el-table-column label="标题长度" width="180">
              <template slot-scope="scope">
                {{scope.row.title.length}}
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
      <!-- 过滤区 -->
      <div class="filters">
        <div class="left">
          <div class="txt">
            <textarea v-model="keyword" @change="change"></textarea>
            <div :class="tableOK?'ok done':'ok'">采集完成</div>
          </div>
          <div class="radio">
            <div class="top">
              <el-radio v-model="radio" label="1">综合</el-radio>
              <el-radio v-model="radio" label="2">销量</el-radio>
              <el-radio v-model="radio" label="3">信用</el-radio>
            </div>
            <div class="price">
              <el-checkbox v-model="checked">价格：</el-checkbox>
              <input type="text" v-model="price1">
              <span class="line"> -</span>
              <input type="text" v-model="price2">
            </div>
            <div class="page">
              <div>采集页数：</div>
              <input type="text" v-model="page">
            </div>
            <div class="buttons">
              <el-button size='small' @click="search">开始采集</el-button>
              <el-button size='small'>导出</el-button>
              <el-button size='small'>添加标题</el-button>
            </div>

          </div>
        </div>
        <div class="right">
          <div class="keys">
            <textarea v-model="keyFilter"></textarea>
            <div class="radio">
              <el-radio v-model="keyRadio" :label="1">只删除关键词</el-radio>
              <el-radio v-model="keyRadio" :label="2">删除整个标题</el-radio>
            </div>
          </div>
          <div class="btns">
            <el-button size='small'>更新关键词</el-button>
            <el-button size='small'>提交关键词</el-button>
            <el-button size='small'>保存关键词</el-button>
            <el-button size='small'>过滤关键词</el-button>
          </div>
        </div>
      </div>
      <!-- 筛选区 -->
      <div class="choose">
        <div class="c c1">
          <el-select v-model="likeValue" placeholder="请选择" size="mini">
            <el-option v-for="i in likeOptions" :key="i.value" :label="i.label" :value="i.value">
            </el-option>
          </el-select>
          <div class="btn">过滤相似度过高的标题</div>
          <p>建议选择过滤0.96以上的相似度</p>
        </div>
        <div class="c c2">
          <div class="btn">过滤分裂标题品牌词</div>
          <p>品牌不全，过滤后请人工检查</p>
        </div>
        <div class="c c3">
          <div class="btn">分裂标题关键词替换</div>
          <div class="replace">
            <input type="text" :value="keyReplace1">
            <span>替换成</span>
            <input type="text" :value="keyReplace2">
          </div>
        </div>
        <div class="c c4">
          <div class="btn">删除标题超出60字节部分</div>
          <div class="del">
            <div class="btn">删除</div>
            <input type="text" :value="del">
            <span>字节以下标题</span>
          </div>
        </div>
      </div>
      <!--  -->
      <webview id="foo" src="https://www.baidu.com" allowpopups></webview>
    </main>·
  </div>
</template>

<script>
export default {
  name: "landing-page",
  data() {
    return {
      tableData: [],
      loading: 0,
      // filters
      keyword: "被子",
      tableOK: 0,
      radio: "1",
      checked: true,
      price1: "1",
      price2: "99999",
      page: "1",
      // keys
      keyFilter: "南极人|水星家纺",
      keyRadio: 1,
      // 相似度
      likeOptions: [
        {
          value: "0.1",
          label: "相似度10%"
        },
        {
          value: "0.2",
          label: "相似度20%"
        },
        {
          value: "0.3",
          label: "相似度30%"
        },
        {
          value: "0.4",
          label: "相似度40%"
        },
        {
          value: "0.5",
          label: "相似度50%"
        },
        {
          value: "0.6",
          label: "相似度60%"
        },
        {
          value: "0.7",
          label: "相似度70%"
        },
        {
          value: "0.8",
          label: "相似度80%"
        }
      ],
      likeValue: "0.1",
      // 关键词替换
      keyReplace1: "",
      keyReplace2: "",
      // 删除
      del: ""
    };
  },
  created() {
    const webview = document.getElementById("foo");
    webview.addEventListener("new-window", e => {
      const protocol = require("url").parse(e.url).protocol;
      if (protocol === "http:" || protocol === "https:") {
        webview.src = e.url;
      }
    });
  },
  methods: {
    search() {
      this.loading = 1;
      let url = `gettitle?keyword=${this.keyword}&page=${this.page}`;
      let res;
      let that = this;
      this.http.get(url).then(res => {
        console.log(res);
        let data = res.data.data;
        console.log("过滤前", data);
        // 过滤 - 删除标题
        if (this.keyRadio === 2)
          data = data.filter(i =>
            this.keyFilter.split("|").every(j => i.title.indexOf(j) === -1)
          );
        // 过滤 - 删除关键字
        if (this.keyRadio === 1)
          data.map(i =>
            this.keyFilter
              .split("|")
              .forEach(j => (i.title = i.title.replace(j, "")))
          );

        console.log("过滤后：", data);
        this.tableData = data;
        this.loading = 0;
      });
    },
    open(link) {
      console.log(1);
      require("electron").shell.openExternal(link);
    },
    index(i) {
      return i + 1;
    },
    change() {
      console.log(this.filter);
      if (this.filter !== "") {
        this.tableOK = 1;
        return;
      }
      this.tableOK = 0;
    }
  }
};
</script>

<style lang='stylus'>
mainC = #409EFF
*
  box-sizing border-box
  margin 0
  padding 0
webview
  height 500px
input, textarea
  outline 0
input
  padding-left 6px
body
  font-family 'Source Sans Pro', sans-serif
  background #eee
  font-size 12px
main
  display flex
  padding 20px
  flex-flow column
.titles
  width 100%
  .el-table
    min-height 200px
    max-height 600px
    overflow auto
    font-size 12px
.filters
  display flex
  flex-flow row
  margin-top 10px
  .left
    flex 3
    background #fff
    margin-right 10px
    display flex
    flex-flow row
    padding 10px
    .txt
      flex 3
      display flex
      flex-flow column
      position relative
      overflow hidden
      textarea
        flex 1
        width 95%
        resize none
        border 1px solid #ccc
        padding 6px
        border-radius 4px
      .ok
        position absolute
        left 0
        bottom -30px
        width 95%
        height 30px
        line-height 30px
        text-align center
        background mainC
        color #fff
        transition 0.3s
        opacity 0
      .ok.done
        bottom 0
        opacity 1
    .radio
      flex 2
      input
        height 26px
        line-height 26px
        width 60px
      .top
        margin-bottom 6px
      .price
        display flex
        flex-flow row nowrap
        align-items center
        .line
          margin 0 6px
      .page
        display flex
        flex-flow row
        align-items center
        margin 6px 0
        div
          font-size 14px
          color #666
    .buttons
      display flex
      flex-flow row nowrap
  .right
    padding 10px
    flex 2
    background #fff
    display flex
    flex-flow row
    .keys
      flex 3
      display flex
      flex-flow column
      textarea
        flex 3
        width 95%
        resize none
        border 1px solid #ccc
        padding 6px
        border-radius 4px
      .radio
        flex 1
        margin-top 8px
        display flex
        align-items center
    .btns
      flex 1
      display flex
      flex-flow column
      .el-button
        flex 1
        margin 3px 0
.choose
  display flex
  flex-flow row nowrap
  margin 0 -5px
  .c
    background #ffffff
    flex 1
    width 25%
    padding 6px
    margin 10px 5px
    .el-input--mini .el-input__inner, .el-input--mini .el-input__icon
      height 25px
      line-height 25px
    .btn
      width 100%
      height 25px
      line-height 25px
      background mainC
      color #ffffff
      text-align center
      border-radius 4px
      transition 0.3s
      cursor pointer
      opacity 0.85
    .btn:hover
      opacity 1
    p
      font-size 10px
  .c1
    .btn
      margin 5px 0
  .c2
    .btn
      height 50px
      line-height 50px
      margin-bottom 10px
  .c3
    .btn
      height 50px
      line-height 50px
      margin-bottom 6px
    .replace
      display flex
      flex-flow row
      align-items center
      justify-content center
      input
        width 33%
        height 25px
        border-radius 5px
      span
        width 33%
        text-align center
  .c4
    .btn
      height 30px
      line-height 30px
      margin-bottom 12px
    .del
      display flex
      flex-flow row nowrap
      align-items center
      justify-content center
      input
        width 30%
        height 25px
        border-radius 5px
        margin 0 2%
      div
        width 30%
        margin-bottom 0
      span
        width 40%
        // font-size 10px
</style>
