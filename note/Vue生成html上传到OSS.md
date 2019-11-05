```js
async handlePublish() {
      this.loading = true
      let html = '<!DOCTYPE html><html><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0"></head><body>' + this.content + '</body></html>'
      const script = '\<script src="http://jialin-website.oss-cn-shenzhen.aliyuncs.com/static/jquery/jquery-3.3.1.min.js"\>\<\/script><script>$(function () {$("iframe").attr("width", document.body.clientWidth);$("iframe").attr("height", (document.body.clientWidth * 9) / 16);$("img").css("maxWidth", document.body.clientWidth);})\<\/script>'
      html = html.replace('https', 'http')
      const content = html + script
      let url = ''
      const uuid = require('node-uuid')
      await this.handleGetOSSConfig({ uid: '-1' })
      this.ossConfig.objectName = 'web/leifengbang/leifengbangFile/' + uuid.v1() + '.html'
      this.ossConfig.fileType = '~leifengFile'
      const OSS = require('ali-oss')

      const client = new OSS({
        region: 'oss-cn-shenzhen',
        // 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用STS方式来进行API访问
        accessKeyId: this.ossConfig.accessKeyId,
        accessKeySecret: this.ossConfig.accessKeySecret,
        stsToken: this.ossConfig.securityToken,
        bucket: this.ossConfig.bucketName
      })
      async function putBlob(objectName, content) {
        try {
          const result = await client.put(objectName, new Blob([content], { type: 'text/html' }))
          url = result.url
          url = url.replace('https', 'http')
        } catch (e) {
          console.log(e)
        }
      }
      await putBlob(this.ossConfig.objectName, content)
      
      publish(this.title,
        this.periodsValue,
        this.listIMG,
        [{ type: 'TXT', content: url, sort: 0 }])
        .then(response => {
          if (response.errorCode === '00') {
            rate(parseInt(response.data), this.rate).then(res => {
              if (res.errorCode === '00') {
                this.$message({
                  message: '发布成功',
                  type: 'success'
                })
              }
            })
          }
          this.loading = false
        })
    }
```