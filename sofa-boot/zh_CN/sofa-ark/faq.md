## 常见问题

### 咨询类
#### Q: 类隔离在蚂蚁内部使用是否广泛？

类隔离在蚂蚁内部使用非常广泛，绝大部分业务应用都是运行在蚂蚁中间件自研的类隔离框架之上。主要是为了解决依赖冲突的问题，像蚂蚁金服这种体量的公司，业务线繁杂、基础服务组件众多，很难做到对所有 JAR 包做统一管控。特别涉及到跨团队模块组件相互依赖时，因为各自技术栈历史包袱的存在，难以有效统一冲突包版本。使用类隔离技术解决了实际开发中的很多痛点，业务开发者不需要担心自身依赖冲突的问题，在多团队协作开发中，也有很大的优势。

#### Q: SOFABoot类隔离框架（SOFAArk）和 OSGI 容器有哪些差异？

作为开源界早负盛名的动态模块系统，基于 OSGi 规范的 Equinox、Felix 等同样具备类隔离能力，然而他们更多强调的是一种编程模型，面向模块化开发，有一整套模块生命周期的管理，定义模块通信机制以及复杂的类加载模型。作为专注于解决依赖冲突的隔离框架，SOFAArk 专注于类隔离，简化了类加载模型，因此显得更加轻量。其次在 OSGi 规范中，所有的模块定义成 Bundle 形式，作为应用开发者，他需要了解 OSGi 背后的工作原理，对开发者要求比较高。在 SOFAArk 中，定义了两层模块类型，Ark Plugin 和 Ark Biz，应用开发者只需要添加隔离的 Ark Plugin 依赖，底层的类加载模型对应用开发者俩说是透明的，基本不会带来额外的学习成本。

### Q: SOFAArk 和 Java9 模块化有哪些差异？
Jigsaw 作为 Java9 模块化方案，抛开内部实现细节，在使用规范上和 OSGi 特别相似：模块的依赖、包导入导出、动态导出、可读性传递、模块服务注册与消费、开放模块、可选模块等等若干概念，相对于 SOFAArk 简单的包导入导出显然过于复杂。在实现细节上，考虑到 JDK 代码的兼容性，Jigsaw 没有采用类加载器隔离的方式，不同模块之间仍然可能是同一个类加载器加载。严格上来讲，Jigsaw 并没有解决同一个类多版本的问题，但是因为模块显示的依赖声明，使用纯 Jigsaw 模块化编程，不同版本类冲突的问题在编译期就能被检查或者启动失败，因为不允许不同模块含有相同类名的包。对于在实际开发中遇到的一类情况，例如两个组件依赖不同版本 hessian 包，即使这两个组件定义成了两个模块，运行时也只有一个hessian版本被加载，依然解决不了不同版本类共存的问题。另外，Jigsaw 相对 Ark 或者 OSGi 有一个明显的缺点，Jigsaw 不允许运行时动态发布模块服务，模块间的通信依赖在 module-info.java 中使用 provides 和 uses 静态注册和引用模块服务。当然，Jigsaw 有很多自己的优点，通过引入module-path，在 module 中显示声明模块依赖关系，避免了传统 maven/gradle 中因为间接依赖导致运行时加载类不确定的缺点；其次通过设置模块包的导入导出配置，可以完全做到接口和实现的分离，提升安全性；另外 Java9 本身借助模块化改造，使用jlink工具，开发者可以将自身应用必须的模块聚合，打包一个自定义的jre镜像。

### 使用类
#### Q: 为什么使用 SNAPSHOT 版本拉取不到依赖？
如果需要使用处于研发状态的 SNAPSHOT 版本，有两种方式：1、拉取 sofa-ark 仓库代码，本地执行 `mvn install`。2、在本地 maven setting.xml 文件增加如下 profile 配置:
```xml
<profile>
    <id>default</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <repositories>
        <repository>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
            <id>maven-snapshot</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
            <id>maven-snapshot</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </pluginRepository>
    </pluginRepositories>
</profile>
```

#### Q: 为什么使用 java -jar 启动 Spring Boot/SOFABoot 应用 Ark 包时，应用自动退出？
因为 SOFAArk 容器不会开启任何非 Daemon 线程，如果是非 Web 应用或者应用启动时不会创建非 Daemon 线程，则应用在执行完 main 方法时，会正常退出。判断 Ark 包是否正常启动，可以观察是否有如下日志出现：
```text
Ark container started in xxx ms.
```