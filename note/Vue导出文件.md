1. 导出excel文件
```html
<el-button
  size="medium"
  type="primary"
  @click="exportExcel()">导出报告</el-button>
  
  --------------------------------------------------------
  
  <el-table
      :data="task_list"
      header-align=“center”
      :header-cell-style="handleOptionTableHeaderStyle"
      id="out-table">
```
```js
import FileSaver from 'file-saver'
import XLSX from 'xlsx'

 exportExcel() {
       var wb = XLSX.utils.table_to_book(document.querySelector('#out-table'))
       var wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: true, type: 'array' })
       try {
         FileSaver.saveAs(new Blob([wbout], { type: 'application/octet-stream' }), 'sheetjs.xlsx')
       } catch (e) { if (typeof console !== 'undefined') console.log(e, wbout) }
       return wbout
     },
```
2. 导出html文件
```html
<el-button
  size="medium"
  type="primary" @click="download('导出报告')">&nbsp;&nbsp;&nbsp;&nbsp;导出报告&nbsp;&nbsp;&nbsp;&nbsp;</el-button>
-------------------------------------------
 <el-table
      :data="task_list"
      header-align=“center”
      :header-cell-style="handleOptionTableHeaderStyle"
      id="resume"
      ref="resume">
```
```js
import { resumecss } from '@/styles/download.css.js'
function writer(fileName, content, option) {
  var a = document.createElement('a')
  var url = window.URL.createObjectURL(new Blob([content],
    { type: (option.type || 'text/plain') + ';charset=' + (option.encoding || 'utf-8') }))
  a.href = url
  a.download = fileName || 'file'
  a.click()
  window.URL.revokeObjectURL(url)
}
-------------------------------------------
methods: {
    download(name) {
      const html = this.getHtml()
      const s = writer(`${name}.html`, html, 'utf-8')
    },
    getHtml() {
      console.log(this.$refs)
      const template = this.$refs.resume.$el.innerHTML
      const html = `<!DOCTYPE html>
                 <html>
                 <head>
                 <meta charset="utf-8">
                 <meta name="viewport" content="width=device-width,initial-scale=1.0">
                 <title>任务报告</title>
                 <style> ${resumecss} </style>
                </head>
                <body>
                <div style="margin:0 auto;width:1200px">
                ${template}
                </div>
                </body>
                </html>`
      return html
    },
    
--------------------------------------
//@/styles/download.css.js
export const resumecss = `
*{
  text-align: center;
}
.task-list-box {
  list-style: none;
  text-align: left;
}
.task-title-none{
  font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
  font: 14px Base;
  width: 100%;
  text-align: center;
}
.task-title:hover{
  cursor: pointer;
}
.task-title {
  line-height: 4vh;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  font: 14px;
  font-weight: 500;
}
.task-list-active::before {
  content: ●;
}
.task-list-active {
  color: #fff;
  background: rgb(87, 140, 255);
}
.task-title-text{
  font: 18px large;
}
.task-report-title-text{
  font: 14px Base;
  text-align: left;
}
.task-point-title-text{
  font: 13px Extra Small;
  font-weight: bold;
}
.task-point-tip-text{
  font: 12px Extra Extra Small;
}
`

```
3. 导出PDF文件
```js
// src/utils/htmlToPdf.js
// 下面两个package要单独安装
import html2Canvas from 'html2canvas'
import JsPDF from 'jspdf'

export default{
  install (Vue, options) {
    Vue.prototype.getPdf = function (id,title) {
      html2Canvas(document.querySelector(`#${id}`), {
        // allowTaint: true
        useCORS:true//看情况选用上面还是下面的，
      }).then(function (canvas) {
            let contentWidth = canvas.width
            let contentHeight = canvas.height
            let pageHeight = contentWidth / 592.28 * 841.89
            let leftHeight = contentHeight
            let position = 0
            let imgWidth = 592.28
            let imgHeight = 592.28 / contentWidth * contentHeight
            let pageData = canvas.toDataURL('image/jpeg', 1.0)
            let PDF = new JsPDF('', 'pt', 'a4')
            if (leftHeight < pageHeight) {
                PDF.addImage(pageData, 'JPEG', 0, 0, imgWidth, imgHeight)
            } else {
            while (leftHeight > 0) {
                  PDF.addImage(pageData, 'JPEG', 0, position, imgWidth, imgHeight)
                  leftHeight -= pageHeight
                  position -= 841.89
                  if (leftHeight > 0) {
                      PDF.addPage()
                  }
              }
            }
            PDF.save(title + '.pdf')
        }
      )
    }
  }
}
```
main.js加入代码：
```js
import htmlToPdf from '@/utils/htmlToPdf'
Vue.use(htmlToPdf)
```
需要用到的页面：
```html
<el-button
  size="medium"
  type="primary" @click="getPdf()">&nbsp;&nbsp;&nbsp;&nbsp;导出报告&nbsp;&nbsp;&nbsp;&nbsp;</el-button>
-------------------------------------------
 <el-table
      :data="task_list"
      header-align=“center”
      :header-cell-style="handleOptionTableHeaderStyle"
      id="pdfDom">
```
```js
export default {
  data () {
      return {
      htmlTitle: '页面导出PDF文件名'
      }
  }
 }
```