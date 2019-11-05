1. 下包: ```npm install wangeditor```
2. 组件中配置一下
```
//@components/Wangeditor/index.vue
<template>
  <div id="wangeditor">
    <div ref="editorElem" style="text-align:left;"></div>
  </div>
</template>
<script>
import E from 'wangeditor'
// import { getToken } from '@/utils/auth'
export default {
  name: 'Editor',
  data() {
    return {
      editor: null,
      editorContent: ''
    }
  },
  // catchData是一个类似回调函数，来自父组件，当然也可以自己写一个函数，主要是用来获取富文本编辑器中的html内容用来传递给服务端
  props: ['catchData'], // 接收父组件的方法
  mounted() {
    this.editor = new E(this.$refs.editorElem)
    // 编辑器的事件，每次改变会获取其html内容
    this.editor.customConfig.onchange = html => {
      this.editorContent = html
      this.catchData(this.editorContent) // 把这个html通过catchData的方法传入父组件
    }
    this.editor.customConfig.menus = [
      // 菜单配置
      'head', // 标题
      'bold', // 粗体
      'fontSize', // 字号
      'fontName', // 字体
      'italic', // 斜体
      'underline', // 下划线
      'strikeThrough', // 删除线
      'foreColor', // 文字颜色
      'backColor', // 背景颜色
      'link', // 插入链接
      'list', // 列表
      'justify', // 对齐方式
      'quote', // 引用
      'emoticon', // 表情
      'image', // 插入图片
      'table', // 表格
      'code', // 插入代码
      'undo', // 撤销
      'redo' // 重复
    ]
    this.editor.customConfig.uploadImgShowBase64 = true
    // this.editor.customConfig.uploadImgServer = process.env.BASE_API + '/api/web/mall/ossAddFile'
    // this.editor.customConfig.uploadImgHeaders = {
    //   'Authorization': getToken()
    // }
    // this.editor.customConfig.uploadImgParams = {
    //   ossConfig: {
    //     accessKeyId: '',
    //     accessKeySecret: '',
    //     expiration: '',
    //     securityToken: '',
    //     objectName: '',
    //     fileType: '',
    //     bucketName: '',
    //     imageStyle: 'image/resize,p_50'
    //   }
    // }
    this.editor.create() // 创建富文本实例
  }
}
</script>
   ```
3. 要使用的页面引入该组件
```
<template>
    <wangeditor :catchData="catchData"></wangeditor>
</template>
<script>
import wangeditor from '@/components/Wangeditor'
components: { wangeditor },
methods: {
    catchData(value) {
      this.postData.content = value // 在这里接受子组件传过来的参数，赋值给data里的参数
    },
}
</script>
```
