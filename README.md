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

### velocity模板加载输出过程
<img src="https://github.com/zhaocancsu/content/blob/master/liucheng.png" width="600" />

```Java
            //1.初始化配置
            Velocity.init("velocity.properties");

            //2.创建context，存放变量
            VelocityContext context = new VelocityContext();
            Person person = new Person();
            person.setName("test");
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
### 渲染流程
* 根据模板文件(.vm文件)中定义的语法内容解析成一个抽象语法树(AST)
* 从root节点通过深度优先搜索的方式遍历节点进行渲染处理
* 不同类型子节点渲染方法不一样(render方法实现的不一样)

根节点
```Java
public boolean render( InternalContextAdapter context, Writer writer)
        throws IOException, MethodInvocationException, ParseErrorException, ResourceNotFoundException
    {
        int i, k = jjtGetNumChildren();

        for (i = 0; i < k; i++)
            jjtGetChild(i).render(context, writer);

        return true;
    }
```
文本节点
```Java
public boolean render( InternalContextAdapter context, Writer writer)
        throws IOException
    {
        writer.write(ctext);
        return true;
    }
```
例子，如果模板文件内容如下<br>
```
<html>
   <div>$!{person.sayHello()}:$!{person.name}</div>
</html>
```
<img src="https://github.com/zhaocancsu/content/blob/master/nodes.png" width="1000" />
解析生成的AST,如下

<img src="https://github.com/zhaocancsu/content/blob/master/ast.jpg" width="600" />

ASTReference.ASTMethod.execute
```Java
VelMethod method = ClassUtils.getMethod(methodName, params, paramClasses, 
            o, context, this, strictRef);
        if (method == null) return null;

        try
        {
            /*
             *  get the returned object.  It may be null, and that is
             *  valid for something declared with a void return type.
             *  Since the caller is expecting something to be returned,
             *  as long as things are peachy, we can return an empty
             *  String so ASTReference() correctly figures out that
             *  all is well.
             */

            Object obj = method.invoke(o, params);

            if (obj == null)
            {
                if( method.getReturnType() == Void.TYPE)
                {
                    return "";
                }
            }

            return obj;
        }
        catch( InvocationTargetException ite )
        {
            return handleInvocationException(o, context, ite.getTargetException());
        }
```
ASTReference.ASTIdentifier.execute
```Java
VelPropertyGet vg = null;

        try
        {
            /*
             *  first, see if we have this information cached.
             */

            IntrospectionCacheData icd = context.icacheGet(this);

            /*
             * if we have the cache data and the class of the object we are
             * invoked with is the same as that in the cache, then we must
             * be allright.  The last 'variable' is the method name, and
             * that is fixed in the template :)
             */

            if ( icd != null && (o != null) && (icd.contextData == o.getClass()) )
            {
                vg = (VelPropertyGet) icd.thingy;
            }
            else
            {
                /*
                 *  otherwise, do the introspection, and cache it.  Use the
                 *  uberspector
                 */

                vg = rsvc.getUberspect().getPropertyGet(o,identifier, uberInfo);

                if (vg != null && vg.isCacheable() && (o != null))
                {
                    icd = new IntrospectionCacheData();
                    icd.contextData = o.getClass();
                    icd.thingy = vg;
                    context.icachePut(this,icd);
                }
            }
        }

        /**
         * pass through application level runtime exceptions
         */
        catch( RuntimeException e )
        {
            throw e;
        }
        catch(Exception e)
        {
            String msg = "ASTIdentifier.execute() : identifier = "+identifier;
            log.error(msg, e);
            throw new VelocityException(msg, e);
        }

       .......

        /*
         *  now try and execute.  If we get a MIE, throw that
         *  as the app wants to get these.  If not, log and punt.
         */
        try
        {
            return vg.invoke(o);
        }
        catch(InvocationTargetException ite)
        {
        .......
        }
```

如果对于同一个对象的同一个属性或方法重复多次调用，可考虑使用临时变量保存<br>
#set($name=$!person.name)

## rose中velocity的使用方式
* 在resources中定义velocity.properites
* 模板文件要放在"/views/模块名/"下
* 变量信息通过Invocation.addModel(key,value)设置
* controller中返回要渲染的模板名称(不带.vm)
