# 为什么会用到kong

起因是一个项目之下涉及到多个应用， 然后需要暴露同意的路由地址给前端使用，这时候需要用到一个网关来转发和控制系统之间互相访问的权限。

# kong能做什么

这里先放一个官网的链接[kong官网](https://konghq.com/)

它是一个基于`nginx`的一个lua应用

- 负载均衡
- 权限控制
- 日志
- 等等（去探索把）

下面给张插件图

![我们随便给个插件图看一下](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32bb74f187a34feaa5f677a720e5e2fb~tplv-k3u1fbpfcp-zoom-1.image)

# 安装

[给个链接敷衍一下](https://konghq.com/get-started/#install)

只需要一些简单的`lua`知识就可以二开哦， 作者大大在`jwt`中融入的`redis`来判断用户权限是否更新了

# 使用官方推荐的工具

[源码](https://github.com/Kong/kong)

[构建脚本](https://github.com/Kong/kong-build-tools)
