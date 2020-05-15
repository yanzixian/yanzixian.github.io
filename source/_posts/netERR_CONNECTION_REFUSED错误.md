#### net::ERR_CONNECTION_REFUSED错误

今天vue使用axios发送请求出现了`net::ERR_CONNECTION_REFUSED`错误

排查的时候发现自己配置的前端项目运行的接口为`8080`，

```js
// Various Dev Server settings
        host: 'localhost', // can be overwritten by process.env.HOST
        port: 8080, // can be overwritten by process.env.PORT, if port is in use, a free one will be determined
        autoOpenBrowser: false,
        errorOverlay: true,
        notifyOnErrors: true,
        poll: false, // https://webpack.js.org/configuration/dev-server/#devserver-watchoptions
```

配置了跨域后发送请求，在浏览器中却出现了下面这种情况：

![image-20200511104459602](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200511104501.png)

这个`8848`是从哪儿冒出来的？不应该是`8080`吗？

无论怎样试，`Request URL`中的端口始终是`8848`，索性将`port`更改为`8848`，然后。。。，就可以了

？？？？？

什么意思？

![](https://gitee.com/yanzixian/picBed/raw/master/img202005/20200511105216.png)

有时间得研究一下