1.首先下包
```
npm install -S file-saver
npm install -S xlsx
npm install -D script-loader
```
2.Blob.js 和 export2Excel.js文件下载并修改export2Excel.js
```js
//main.js

import Blob from '@/utils/Blob'
```
```js
//export2Excel.js  Blob文件的位置要修改
require('script-loader!file-saver')
require('./Blob.js')
require('script-loader!xlsx/dist/xlsx.core.min')
```
3.el-table存入当前选择的行属性
```html
<el-table :data="list" v-loading="listloading" size="medium" @selection-change="handleSelectionChange">
```
```js
data() {
    return {
      multipleSelection: []
    }
  },
  
  ------------------------------
    handleSelectionChange(e) {
      this.multipleSelection = e
    },
```
4.配置导出方法
```js
    export2Excel(e) {
      console.log(this.multipleSelection)
      const excelHeader = ['编码']
      const excelFilterVal = ['code']
      require.ensure([], () => {
        const { export_json_to_excel } = require('@/utils/Export2Excel')
        
        const listArr = JSON.parse(JSON.stringify(this.multipleSelection))
        const formatData = this.formatJson(excelFilterVal, listArr)
        export_json_to_excel(excelHeader, formatData, '优惠码导出')
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    }
```
5.勾选完需要的数据后点击即可导出表格文件
```html
<el-button type="primary" @click="export2Excel()">导出优惠码</el-button>
```
