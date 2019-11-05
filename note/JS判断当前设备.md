```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>邻里共享</title>
</head>

<body>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
    <script>
        $(function () {
            var u = navigator.userAgent,
                app = navigator.appVersion;
            var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1; //g
            var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
            var isWx = !!/MicroMessenger/i.test(u);
            var IsPC = !/(Android|iPhone|SymbianOS|Windows Phone|iPad|iPod)/i.test(u);


            if (isAndroid) {
                if (isWx) {
                    window.location.href = 'https://a.app.qq.com/o/simple.jsp?pkgname=com.jialin.linlishare'
                } else {
                    window.location.href =
                        'https://jialintechnology.oss-cn-shenzhen.aliyuncs.com/app/linlishare.apk';
                }
            }
            if (isIOS) {
                window.location.href = 'https://zc.pgyer.com/oolL';
            }
   
            // 是PC端
            if (IsPC) {
                window.location.href = 'https://a.app.qq.com/o/simple.jsp?pkgname=com.jialin.linlishare'; 
            }
        });
    </script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!--防止https请求http出现问题-->
    <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
    <title>广州佳邻网络科技有限公司</title>
</head>

<body>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
    <script>
        $(function () {
            var u = navigator.userAgent,
                app = navigator.appVersion;
            var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1; //g
            var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
            var isIPhone = u.indexOf('iPhone') > -1 //是否为iPhone或者QQHD浏览器
            var isIPad = u.indexOf('iPad') > -1 //是否iPad};
            var isWx = !!/MicroMessenger/i.test(u);
            var IsPC = !/(Android|iPhone|SymbianOS|Windows Phone|iPad|iPod)/i.test(u);


            if (isAndroid) {
                if (isWx) {
                    window.location.href = 'http://www.jialinkeji.cn/mobile/index.html'
                } else {
                    window.location.href =
                        'http://www.jialinkeji.cn/mobile/index.html';
                }
            }
            if (isIOS) {
                if (isIPhone) {
                    window.location.href = 'http://www.jialinkeji.cn/mobile/index.html';
                } else if (isIPad) {
                    window.location.href = 'http://www.jialinkeji.cn/index.html';
                }
            }
   
            // 是PC端
            if (IsPC) {
                window.location.href = 'http://www.jialinkeji.cn/index.html'; 
            }
        });
    </script>
</body>
</html>
```