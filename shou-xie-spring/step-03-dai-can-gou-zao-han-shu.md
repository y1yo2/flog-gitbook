---
description: 2021/09/20
---

# Step 03：带参构造函数

## 一、背景

实例化使用 Class 类的 newInstance 方法，不支持带参构造函数。

问题1.使用哪种方式支持带参构造函数？Class类的带参实例化方法是什么？

JDK反射获取带参构造函数，完成实例化。Class类 getConstructor\(Class...\) 获取特定参数的带参构造方法，并使用 Constructor类 newInstance\(Object...\) 完成实例化。

必须使用目标类的Class对象。要求目标类必须存在带参构造函数，且必须清楚该函数的入参类型、顺序。适配性、泛用性差，扩展性差（强耦合）。

问题2.Cglib 通过编译时生成子类的 Class（字节码）文件，继承父类，并包含相应的带参构造函数完成实例化。

Cglib 通过ASM框架，在编译时生成目标类的子类 .Class 文件（字节码）。......（待补充）

3.Class 信息在 JVM 的存放位置，Class 受 final 的影响，final Class 成员变量、成员方法在 JVM 的存放位置，使用限制（影响）。

|  |  |  |
| :--- | :--- | :--- |
| 每个线程都有一个 | 程序计数器 |  |
|  | Java虚拟机栈 | 当前线程执行函数的局部变量表、动态链接、结果、异常...... |
|  | 本地方法栈 |  |
| 共享区 | 堆 | 新生代（A/B），老年代 |
|  | 方法区（元数据区） | 包含各种元数据+对象常量池 |

因此，Class信息在 JDK8后，存放于Hotspot JVM的元数据区。

Final 的影响：

* final 类：唯一作用是，标记不允许被继承。
* final 方法：最重要作用是，标记该方法不允许被继承类重写。
  * 另一个作用是，方法内联，编译时将方法直接嵌入调用的代码块中。新版JVM已自动执行方法内联，无需特意 final标识方法。
* final 成员变量：
* final 局部变量：



## 二、设计

![&#x589E;&#x52A0;Cglib&#x5B9E;&#x4F8B;&#x5316;&#x7B56;&#x7565;&#x7684;Bean&#x5BB9;&#x5668;UML&#x56FE;](../.gitbook/assets/bean-rong-qi-cglib-shi-li-hua-.png)

* AbstractAutowireCapableBeanFactory 中：增加目标类通过有参构造器实例化的逻辑。（该Factory主要负责createBean的实现，体现了每层类负责单一功能，并通过继承获得能力。）
* InstantiationStrategy接口：策略模式，该接口抽象了实例化能力，并通过不同实现类实现各种实例化的逻辑（包括Cglib实例化的逻辑）。

## 三、实现

### 实现类

#### AbstractAutowireCapableBeanFactory

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory{

    private InstantiationStrategy instantiationStrategy = new CglibSubclassInstantiationStrategy();

    @Override
    protected Object createBean(BeanDefinition beanDefinition, String beanName, Object[] args) {
        Object bean = createBeanInstance(beanDefinition, beanName, args);
        putSingletonBean(beanName, bean);
        return bean;
    }

    private Object createBeanInstance(BeanDefinition beanDefinition, String beanName, Object[] args) {
        Class aClass = beanDefinition.getBeanClass();
        Constructor constructor = null;
        Constructor[] cons = aClass.getDeclaredConstructors();
        for (Constructor con: cons) {
            if (args!=null && con.getParameterTypes().length == args.length) {
                constructor = con;
                break;
            }
        }
        return instantiationStrategy.instantiate(beanDefinition, beanName, constructor, args);
    }
}
```

#### InstantiationStrategy

```java
public interface InstantiationStrategy {

    public Object instantiate(BeanDefinition beanDefinition, String beanName,
                              Constructor constructor, Object[] args);

}
```

#### CglibSubclassInstantiationStrategy

```java
public class CglibSubclassInstantiationStrategy implements InstantiationStrategy{

    @Override
    public Object instantiate(BeanDefinition beanDefinition, String beanName, Constructor constructor, Object[] args) {
        Class aClass = beanDefinition.getBeanClass();
        Object result = null;
        try {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(aClass);
            enhancer.setCallback(new NoOp() {});
            if (constructor != null) {
                result = enhancer.create(constructor.getParameterTypes(), args);
            }else {
                result = enhancer.create();
            }
        }catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

拓展知识：

class.getDeclaredConstructors\(\);

class.getConstructors\(\);

constructor.getParameterTypes\(\);





















