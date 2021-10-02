---
description: 2021/10/02
---

# Step 04：依赖注入

一、背景

回顾我们实现的容器，已具备Bean的定义、注册、获取、实例化（带参/无参）的功能，容器既存储BeanDefinition（Bean类型），也存储Bean实例。

上一个需求，是兼容实例化时，带参的构造函数。这次又提新需求，实现实例化后，对象的属性值填充（Spring的核心：依赖注入）。

得益于设计模式的使用，代码保留了良好的扩展性。例如：

* BeanFactory的**工厂模式**抽象了Bean的出生过程（注册、获取、实例化、依赖注入等），统一了Bean的获取入口；
* AbstractBeanFactory的**模板模式**，则将核心功能固定，且通过多层继承，每层实现类实现不同的重点逻辑（DefaultBeanFactory只负责存储和管理BeanDefinition，该能力由BeanDefinitionRegistry接口定义）；
* InstantiationStrategy的**策略模式**，由接口定义实例化能力后，通过不同类实现不同的实例化逻辑（例如CglibInstantiationStrategy 实现Cglib代理实例化的逻辑）。使用类可灵活选择具体策略。

二、设计

Bean实例化后的属性值填充，可分解为两部分：  
1、BeanDefinition需要记录属性值信息；  
2、负责实例化的BeanFactory（AbstractAutowireCapableBeanFactory），要增加处理BeanDefinition中属性信息的逻辑。

因此，类设计思路及UML图如下：

1. 通过 PropertyValues 存放属性名name和属性值object，
2. 当属性值为 BeanReference时，该属性为容器Bean，需要getBean获得属性值，且 BeanReference存放 beanName；

![BeanDefinition&#x6DFB;&#x52A0;&#x5C5E;&#x6027;&#x4FE1;&#x606F;&#x53CA;&#x5904;&#x7406;&#x903B;&#x8F91;UML&#x56FE;](../.gitbook/assets/1633161118-1-%20%281%29.png)

## 三、实现

### 实现类

#### AbstractAutowireCapableBeanFactory

```java
public abstract class AbstractAutowireCapableBeanFactory  extends AbstractBeanFactory {

    private InstantiationStrategy instantiationStrategy = new CglibSubclassInstantiationStrategy();

    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) {
        Object bean = createBeanInstance(beanDefinition, args);
        applyPropertyValues(beanDefinition, bean);
        putSingletonBean(beanName, bean);
        return bean;
    }

    private Object createBeanInstance(BeanDefinition beanDefinition, Object[] args) {
        Class clazz = beanDefinition.getBeanClass();
        Constructor constructor = null;
        for (Constructor cos:clazz.getDeclaredConstructors()) {
            if (args != null && args.length == cos.getParameterCount()) {
                constructor = cos;
                break;
            }
        }
        return instantiationStrategy.instantiate(beanDefinition, constructor, args);
    }

    private Object applyPropertyValues(BeanDefinition beanDefinition, Object bean) {
        PropertyValues pvs = beanDefinition.getPropertyValues();
        try {
            if (pvs != null) {
                for (PropertyValue pv:pvs.getPvs()) {
                    Object propertyValue = pv.getProperty();
                    String propertyName = pv.getName();
                    if (propertyValue instanceof BeanReference) {
                        BeanReference beanReference = (BeanReference) propertyValue;
                        propertyValue = getBean(beanReference.getBeanName());
                    }
                    // Map、List、Array、普通Object
                    // 普通Object：递归获取所有属性
                    BeanUtil.setFieldValue(bean, propertyName, propertyValue);
                }
            }
        }catch (Exception e) {
            e.printStackTrace();
        }
        return bean;
    }

    public void setInstantiationStrategy(InstantiationStrategy instantiationStrategy) {
        this.instantiationStrategy = instantiationStrategy;
    }
}
```

#### BeanDefinition

```java
public class BeanDefinition {

    private Class beanClass;

    private PropertyValues propertyValues;

    public BeanDefinition(Class beanClass) {
        this.beanClass = beanClass;
    }

    public BeanDefinition(Class beanClass, PropertyValues propertyValues) {
        this.beanClass = beanClass;
        this.propertyValues = propertyValues;
    }

    public Class getBeanClass() {
        return beanClass;
    }

    public void setBeanClass(Class beanClass) {
        this.beanClass = beanClass;
    }

    public PropertyValues getPropertyValues() {
        return propertyValues;
    }

    public void setPropertyValues(PropertyValues propertyValues) {
        this.propertyValues = propertyValues;
    }
}
```

#### PropertyValues

```java
public class PropertyValues {

    private List<PropertyValue> pvs = new ArrayList<>();

    public PropertyValues() {
    }

    public PropertyValues(List<PropertyValue> pvs) {
        this.pvs = pvs;
    }

    public List<PropertyValue> getPvs() {
        return pvs;
    }

    public void setPvs(List<PropertyValue> pvs) {
        this.pvs = pvs;
    }

}
```

#### PropertyValue

```java
public class PropertyValue {

    private String name;

    private Object property;

    public PropertyValue(String name, Object property) {
        this.name = name;
        this.property = property;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Object getProperty() {
        return property;
    }

    public void setProperty(Object property) {
        this.property = property;
    }
}
```

#### BeanReference

```java
public class BeanReference {

    private final String beanName;

    public BeanReference(String beanName) {
        this.beanName = beanName;
    }

    public String getBeanName() {
        return beanName;
    }
}
```



