<template>
  <div id="app" style="height:100%">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 40,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* Inspired by http://strml.net/
* 大家好，我是许得龙
* 身在帝都，压力大，容不得休息呀，
* 我目前正在找前端开发相关的工作，
* 想找工作，得先有一份简历才行
* 说做就做，马上写一份简历！
* 希望能和正在看简历的帅哥或者美女成为同事
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all 1s;
}
/* 白色背景太单调了，我们来点背景 */
html,body{
  color: rgb(222,222,222); background: rgb(63, 82, 99);
  height: 100%;
  overflow: hidden;
  font-family: "Helvetica Neue", Helvetica, sans-serif
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 50%;
  max-height: 98%;
  background:rgb(48, 48, 48)
}
/* 代码高亮 */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 加点 3D 效果呗 */
html{
  perspective: 1000px;
}
.styleEditor {
  -webkit-transform: rotateY(20deg);
  -webkit-transform-origin: left;
}

/* 接下来我给自己准备一个编辑器 */
.resumeEditor{
    position: absolute;
    padding: .5em;
    margin: .5em;
    left: 50%;
    top: 0px;
    width: 50%;
    height: 98%;
    border: 1px solid;
    background: white;
    color: #222;
    overflow: auto;
    border: 1px solid;
    -webkit-transform: rotateY(-20deg);
    -webkit-transform-origin: right;
}
/* 好了，我开始写简历了 */


`,
          `
/* 这个简历好像差点什么
 * 对了，这是 Markdown 格式的，我需要变成对 HR 更友好的格式
 * 简单，用开源工具翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */

.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
.resumeEditor {
  -webkit-transform: rotateY(0deg);
}
.styleEditor {
  -webkit-transform: rotateY(0deg);
}
`],
        currentMarkdown: '',
        fullMarkdown: `基本信息
----

许得龙 / 男 / 26岁 / 已婚  /  居住北京  /  前端工程师  /  工作经验 4年

技能
----

* 前端开发(vue.js angular.js)
* android开发
* Node.js 开发(了解)
* php后台开发

工作经历
----

1. 北京景蓝信息技术有限公司（2015.2 - 2018.1）职位：合伙人
2. 百度（2014.7 - 2015.2）职位：外派前端开发（有机会转正）

链接
----

* [GitHub](https://github.com/delonglimin)
* [我的博客](https://xudelong.top)(练手的项目)

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
