---
description: 2021/09/15
---

# Step 01：实现简单容器

## 一、背景

当我们使用面向对象编程语言，拥有 封装、继承、多态，三板斧。  
进行软件工程建设时，一般需要对工作、功能进行抽象，形成工作类，并创建、管理、使用他们。  
例如，在控制类的成员方法中，new 工作类并调用工作类对象的方法。

产生如下问题：  
1.控制类直接使用具体的工作类，与工作类强耦合。（改进：调用工作类的抽象，如接口/抽象类。即控制反转）  
2.需要在每个控制类，编写重复代码描述工作类的创建对象。（改进：统一控制类获得对象的方式。即依赖注入）  
3.每个工作类的对象创建、管理方式不同，可能是单例、每次new 、根据控制类 new等，且需要在每个工作类中编写创建、管理的代码/说明。

因此，我们将工作类统一管理，将类、对象都放入容器，形成对象池（池化思想），需要相应类的对象时，从池中获取对象。

将控制类也统一管理，自动为控制类注入依赖的对象，避免每个控制类都编写从容器获取对象的重复代码。

（为什么需要Spring容器？）

## 二、设计

设计一个 Spring Bean容器，首先，容器需要存放Bean，且能根据Bean的标识获取指定的Bean。数据结构选型为 HashMap（基于扰动函数、负载因子、红黑树转换、拉链寻址的数据结构，数据读取的时间复杂度为O\(1\)~ O\(Logn\) ~O\(n\)）。

其次，Bean容器 还包括Bean的定义、注册、获取三个功能。根据数据结构选型和具体功能，工程设计和UML图如下：

* 定义：BeanDefinition，存放Bean的类信息，用于创建Bean对象。本次直接存放Bean对象（使用Object）。
* 注册：BeanFactory，工厂模式，统一由工厂类管理对象的注册和获取。注册对象，registerBean（使用HashMap）。
* 获取：BeanFactory，Spring Bean容器初始化完成后，可获取对象，getBean。

![&#x7B80;&#x5355;&#x5BB9;&#x5668;UML&#x56FE;](../.gitbook/assets/image%20%2820%29.png)

UML图说明：  
1.BeanDefinition：存放定义Bean的实例化信息，暂用Object存放Bean的实例化对象。  
2.BeanFactory：Bean对象的工厂类，存放Bean容器Map，定义Bean的注册和获取方式。

## 三、实现

工程结构

```text
small-spring-step-01
└──src
  ├─main
  │  └─java
  │     └─cn.fanyy51.smallspring
  │         ├─BeanDefinition.java
  │         └─BeanFactory.java    
  └─test
      └─java
          └─cn.fanyy51.smallspring
              ├─bean
              │  └─UserService.java
              └─ControllerTest.java         
```

实现类

cn/fanyy51/smallspring/BeanDefinition.java

```java
public class BeanDefinition {

    private Object bean;

    public BeanDefinition(Object bean) {
        this.bean = bean;
    }

    public Object getBean() {
        return bean;
    }
}
```

cn/fanyy51/smallspring/BeanFactory.java

```java
public class BeanFactory {

    private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();

    public Object getBean(String beanName) {
        return beanDefinitionMap.get(beanName).getBean();
    }

    public void registerBean(String beanName, BeanDefinition beanDefinition) {
        beanDefinitionMap.put(beanName, beanDefinition);
    }
}
```

cn/fanyy51/smallspring/bean/UserService.java

```java
public class UserService {

    public void queryUserInfo() {
        System.out.println("查询用户信息");
    }
}
```

cn/fanyy51/smallspring/ControllerTest.java

```java
@Test
public void testBeanFactory() {
    // 1.初始化 BeanFactory
    BeanFactory beanFactory = new BeanFactory();

    // 2.注册 Bean
    BeanDefinition beanDefinition = new BeanDefinition(new UserService());
    beanFactory.registerBean("userService", beanDefinition);

    // 3.获取 Bean
    UserService userService = (UserService) beanFactory.getBean("userService");
    userService.queryUserInfo();
}
```

测试结果：`查询用户信息` 





