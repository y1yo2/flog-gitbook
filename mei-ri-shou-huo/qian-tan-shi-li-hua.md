---
description: 2021/09/22
---

# 浅谈实例化

## 一、背景

模仿Spring源码时，需考虑 bean如何实例化。在日常工作中都是通过 new调用构造函数，但该方式扩展性不足，与相关类强耦合。

那么使用Java如何完成通用的实例化工具？首先我们必须将“类”抽象为一个类，即 Class类。

## 二、实现方法

在 Class类中直接带有实例化方法，newInstance\(\)。注意该方法从JDK9被标记为废弃的 @Deprecated\(since="9"\)。

### 带参数的构造函数

newInstance无法满足带参数构造函数。常见的实践有以下三种：

#### JAVA反射

通过 Class.getConstructors\(\) 或 getDeclaredConstructors\(\) 获取构造器，通过 Constructors.newInstance\(args\) 调用带参构造函数实例化。

> getConstructors：类本身的public构造器，父类的public构造器
>
> getDeclaredConstructors：类本省的所有修饰的构造器
>
> 如何获取父类的非public构造器？
>
> getSubClass\(\).getDeclaredConstructors\(\) 即可。

#### Cglib

通过 ASM框架，生成继承目标类的 .Class文件（字节码文件），得到继承目标类的子类（增强类）。









