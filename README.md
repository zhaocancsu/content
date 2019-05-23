# 服务端渲染(SSR)

## 什么是SSR(server side render)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;服务端不仅进行数据的获取、转换和处理操作，同时也负责页面的生成，最终传到客户端(浏览器)的是生成的包含数据的页面。客户端所需要做的仅仅是html页面的展现和之后的DOM事件处理。
## SSR解决了什么？
* 前端加载快：前端接收到的数据，已是完全的可视化的页面
* 数据部分获取快速，服务端一般都是内网请求获取数据，相比前端走http外网获取数据，网络开销小

## 服务端渲染 VS 客户端渲染
<img src="https://github.com/zhaocancsu/content/blob/master/0523-2.png" width="600" />
<img src="https://github.com/zhaocancsu/content/blob/master/0523-1.png" width="600" />
