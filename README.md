# 服务端渲染(SSR)

## 什么是SSR(server side render)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;服务端不仅进行数据的获取、转换和处理操作，同时也负责页面的生成，最终传到客户端(浏览器)的是生成的包含数据的页面。客户端所需要做的仅仅是html页面的展现和之后的DOM事件处理。
## SSR解决了什么？
* 前端加载快：前端接收到的数据，已是完全的可视化的页面
* 数据部分获取快速，服务端一般都是内网请求获取数据，相比前端走http外网获取数据，网络开销小

## 服务端渲染 VS 客户端渲染
<img src="https://github.com/zhaocancsu/content/blob/master/0523-2.png" width="600" />
<img src="https://github.com/zhaocancsu/content/blob/master/0523-1.png" width="600" />


java服务端ssr选型

### velocity 
velocity根据view path加载 resolver
```Java
protected ViewResolver getSpringViewResolver(Invocation inv, String viewPath)
            throws IOException {
        if (viewPath.endsWith(".vm")) {
            if (logger.isDebugEnabled()) {
                logger.debug("to get velocity view resolver.");
            }
            return getVelocityViewResolver(inv, viewPath);
        }
    .....
}
```

### velocity模板渲染原理
<img src="https://github.com/zhaocancsu/content/blob/master/liucheng.png" width="600" />

```Java
            //1.初始化配置
            Velocity.init("velocity.properties");

            //2.创建context，存放变量
            VelocityContext context = new VelocityContext();
            Person person = new Person();
            person.setName("jiaduo");
            context.put("person", person);

            //3.加载模板文件到内存
            Template template = null;
            String templateFile = "index.vm";
            template = Velocity.getTemplate(templateFile);

            //4.渲染
            StringWriter stringWriter = new StringWriter();
            template.merge(context, stringWriter);

            //5.打印结果
            System.out.println(stringWriter.toString());
```
