---
description: maven管理多module依赖
---

# Maven统一版本管理

父工程与多个子工程（模块），通过父工程的pom，统一管理所有module的dependency版本。

## 父pom

```markup
<packaging>pom</packaging>

<modules>
  <module>module-one</module>
</modules>
```

**packaging** 打包类型为pom，意思是构建的产物只有pom文件，没有代码需测试或编译，没有资源需处理。（jar、war）

**modules** 表示使用聚合，打包方式必须为pom，module是子模块（相对于当前pom的路径）。

> **module** 的值是要聚合的maven项目相对于该 pom 文件的路径名称，而非 **module** 的 **artifactId。**

### dependencyManagement

**dependencies** 用于引入依赖。**dependencyManagement** 用于指定依赖 的**version**、**scope**、**exclusions**（剔除冲突的依赖），而不引用。

### pluginManagement

同理，pluginManagement用于指定plugin的版本，而不引用。

## 模块pom

```markup
<parent>
    <groupId>com.fan.demo</groupId>
    <artifactId>module</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!--<relativePath/>-->
</parent>

<artifactId>module-one</artifactId>
```

**parent** 表明该pom继承 com.fan.demo:module:0.0.1-SNAPSHOT。

**relativePath** 指定 父项目相对于当前pom的**路径**（默认  `../pom.xml` ）。

> Maven查找父pom，首先是当前构建项目的环境，然后项目所在的文件系统，然后是本地存储库（.m2），最后是远程repo。

 **artifactId** 是子模块的制品id，继承父pom若不写 **groupId**、**version**，默认继承父pom。

## 什么maven聚合，继承？

聚合，通过聚合项目的父pom，配置packaging（pom）、modules。作用，批量管理maven项目，如批量编译、清理、打包（complie、clean、package）

继承，通过pom的 parent 指定父pom。作用，将通用配置项放父pom减少配置。

## type:pom, scope:import

通过父pom管理依赖时，会遇到以下情况：

```markup
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
</dependencyManagement>
```

* **type:pom**，因为该依赖为一个pom项目（名字通常为\*\*\*-dependencies），也是用于管理依赖版本。
* **scope:import**，表示引入该pom项目的 dependencyManagement，而不引入pom中的具体依赖（dependency）。

