---
description: 2021/09/22
---

# 浅谈实例化

## 一、背景

模仿Spring源码时，需考虑 bean如何实例化。在日常工作中都是通过 new调用构造函数，但该方式扩展性不足，与相关类强耦合。

那么使用Java如何完成通用的实例化工具？首先我们必须将“类”抽象为一个类，即 Class类。

## 二、实现方法

在 Class类中直接带有实例化方法，`newInstance()`。注意该方法从JDK9被标记为废弃的 @Deprecated\(since="9"\)。

### 带参数的构造函数

`newInstance()`无法满足带参数构造函数。常见的实践有以下三种：

#### JAVA反射

通过 `Class.getConstructors()` 或 `getDeclaredConstructors()` 获取构造器，通过 `Constructors.newInstance(args)` 调用带参构造函数实例化。

> `getConstructors`：类本身的public构造器，父类的public构造器
>
>  `getDeclaredConstructors`：类本省的所有修饰的构造器
>
> 如何获取父类的非public构造器？
>
>  `getSubClass().getDeclaredConstructors()` ，先获取父类Class对象再获取所有构造器即可。

### Cglib

通过 ASM框架，生成继承目标类的 .Class文件（字节码文件），得到继承目标类的子类（增强类）。

#### 子类（增强类）：

1、从如何生成分析：构建子类需要，目标类的Class、通过callback方法设置的 MethodInterceptor（拦截器入参包括增强类的对象，即this；当前调用的方法Method；方法参数args；Method Proxy）。

2、从增强类的结构分析：增强类继承目标类（无法继承Final类，无法重写Final方法），实现 Factory接口（完成代理功能所需的工具）。

3、从增强类的实现分析：重写方法都标记为Final（无法对代理类再增强）；重写方法逻辑是，有 MethodInterceptor则走拦截器，无则走父类实现。（因此只实例化目标类，可设置为无拦截器或 NoOP）

3.1、MethodInterceptor逻辑：通过 MethodProxy调用增强类方法或父类方法（动态代理核心）。

#### 关于MethodProxy，可分析Cglib动态生成的类

* 增强类
* 两个继承FastClass的匿名内部类

从MethodProxy.init\(\)可知，使用FastClass通过方法的Signature（方法签名，`getSome(Ljava/lang/String;)Ljava/lang/String;`）获取一个 index（每个方法写死一个 int值作为索引）。

当MethodProxy.invoke\(\)时，直接调用FastClass.invoke\(\)，通过Signature获得index，通过index（switch...case...）直接调用相关类的方法。而两个FastClass分别对应目标类和增强类。

因此，MethodProxy和FastClass，通过建立方法签名和index的对应关系，直接调用方法，通过非反射的方式实现方法调用。

最后，每一个MethodProxy实例都是延迟加载，当调用 proxy.invoke\(\)都会执行以下逻辑：

```java
public Object invoke(Object obj, Object[] args) throws Throwable {
  try {
    init();//延迟加载措施
    }
  }
  
  private void init()
{
  if (fastClassInfo == null)
  {
    synchronized (initLock)
    {
      if (fastClassInfo == null)
      {
      //初始化索引和方法签名的关系
      }
    }
  }
}
```





