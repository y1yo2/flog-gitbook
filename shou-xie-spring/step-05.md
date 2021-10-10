---
description: 10/03
---
# Step 05：读取配置

## 一、背景

（补充上一节测试类截图）

```java
@Test
    public void beanReferenceTest() {
        // 1、初始化 BeanFactory
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 2、dao注册
        PropertyValues innerPVs = new PropertyValues();
        innerPVs.getPvs().add(new PropertyValue("inStr", "identify String"));
        BeanDefinition innerBeanDef = new BeanDefinition(InnerImpl.class, innerPVs);
        // 3、service注册，设置属性dao
        PropertyValues userPVs = new PropertyValues();
        userPVs.getPvs().add(new PropertyValue("innerImpl", new BeanReference(InnerImplBeanName)));
        BeanDefinition userBeanDef = new BeanDefinition(UserService.class, userPVs);

        beanFactory.putBeanDefinition(InnerImplBeanName, innerBeanDef);
        beanFactory.putBeanDefinition(UserServiceBeanName, userBeanDef);

        // 4、service获取bean
        UserService userService = (UserService) beanFactory.getBean(UserServiceBeanName);
        userService.queryUserInfo();
        userService.getInnerImpl().queryStr();

    }
```

将 BeanDefinition放入 xml配置文件中，通过读取并解析xml生成 BeanDefinition。通过读取配置完成步骤2、3的动作。



## 二、设计

![读取配置相关UML图](../.gitbook/assets/step05-du-qu-pei-zhi-resource-xiang-guan-.png)

【读取配置部分】

1、通过不同的资源策略（Resource）获得输入流，Resource接口定义获得输入流（资源）的功能。不同实现类完成多种实现逻辑。

一个资源固定为一个Resource对象，因此Resource实现类的属性标记为final，防止创建后更改。getPath等方法标记为final，表示无法被重写。

**接口管定义，抽象类处理非接口功能外的功能/组件填充，实现类可只关心具体的业务实现。**

2、统一用户使用资源（Resource）的姿势，通过资源加载器（ResourceLoader接口）定义获取资源的功能。即ResourceLoader依赖Resource。

3、**统一**使用ResourceLoader读取配置，使用BeanDefinitionRegistry注册Bean定义的**行为**。通过BeanDefinitionReader接口（Bean定义读取接口），定义加载BeanDefinition功能。即BeanDefinitionReader依赖ResourceLoader和BeanDefinitionRegistry。

![BeanFactory接口扩展UML图](../.gitbook/assets/step05-du-qu-pei-zhi-beanfactory-kuo-zhan-.png)

【Bean工厂部分】

回顾之前的设计：

* BeanFactory：定义获取Bean行为；
* AbstractBeanFactory：具有BeanRegistry的能力，实现了getBean功能；
* AbstractAutowireCapableBeanFactory：实现了BeanInstantiation，实例化Bean的功能；
* DefaultListableBeanFactory：实现了BeanDefinition注册、管理的功能；

根据本需求（读取配置注册BeanDefinition），以及后续需求，增加框架扩展性，进行下列拆解设计（参考spring源码）：

* BeanFactory：定义获取Bean行为，增加 getBean(beanName, Class\<T> requiredType)按类型获取（通过泛型解决代码强转问题）；
* ListableBeanFactory：继承BeanFactory的接口，增加管理Bean的功能（getBeansOfType，getBeanDefinitonNames）；
* HierarchicalBeanFactory：分层BeanFactory，增加获取父类BeanFactory从而扩展工厂的层次子接口；
* AutowireCapableBeanFactory：增加自动化处理工厂配置的功能；
* ConfigurableBeanFactory：增加获取BeanPostProcessor，BeanClassLoader等的配置化接口；
* ConfigurableListableBeanFactory：增加分析、修改Bean，预先实例化功能的接口；

## 三、实现

#### Resource

```java
public interface Resource {

    public abstract InputStream getInputStream() throws Exception;

}
```

#### ClassPathResource

```java
public class ClassPathResource implements Resource {

    private final String path;

    private final ClassLoader classLoader;

    @Override
    public InputStream getInputStream() throws Exception {
        InputStream inputStream = classLoader.getResourceAsStream(path);
        if (inputStream == null) {
            throw new FileNotFoundException(this.path + "cannot be opened because file not found");
        }
        return inputStream;
    }
}
```

#### FileSystemResource

```java
public class FileSystemResource implements Resource{

    private final String path;

    private final File file;

    @Override
    public InputStream getInputStream() throws Exception {
        InputStream inputStream = new FileInputStream(file);
        if (inputStream == null) {
            throw new FileNotFoundException(this.path + "cannot be opened because file not found");
        }
        return inputStream;
    }
}
```

#### UrlResource

```java
public class UrlResource implements Resource {

    private final String path;

    private final URL url;

    @Override
    public InputStream getInputStream() throws Exception {
        URLConnection connection = this.url.openConnection();

        try {
            return connection.getInputStream();
        }catch (IOException e) {
            if (connection instanceof HttpURLConnection) {
                 ((HttpURLConnection) connection).disconnect();
            }
            throw e;
        }

    }
}
```

#### ResourceLoader

```java
public interface ResourceLoader {

    static final String CLASSPATH_PREFIX = "classpath:";

    public abstract Resource getResource(String path);
}
```

#### DefaultResourceLoader

```java
public class DefaultResourceLoader implements ResourceLoader{

    @Override
    public Resource getResource(String path) {
        Assert.notNull(path, "Path must be not null");
        if (path.startsWith(CLASSPATH_PREFIX)) {
            // 通过 classpath:前缀，标识使用ClassPathResource，从类路径中加载资源
            return new ClassPathResource(path.substring(CLASSPATH_PREFIX.length()));
        }
        try {
            return new UrlResource(path);
        }catch (MalformedURLException e) {
            return new FileSystemResource(path);
        }

    }
}
```

####

```java
public class DefaultResourceLoader implements ResourceLoader{

    @Override
    public Resource getResource(String path) {
        Assert.notNull(path, "Path must be not null");
        if (path.startsWith(CLASSPATH_PREFIX)) {
            // 通过 classpath:前缀，标识使用ClassPathResource，从类路径中加载资源
            return new ClassPathResource(path.substring(CLASSPATH_PREFIX.length()));
        }
        try {
            return new UrlResource(path);
        }catch (MalformedURLException e) {
            return new FileSystemResource(path);
        }

    }
}
```

#### BeanDefinitionReader

```java
public interface BeanDefinitionReader {

    BeanDefinitionRegistry getBeanDefinitionRegistry();

    ResourceLoader getResourceLoader();

    void loadResource(Resource resource);

    void loadResource(Resource... resources);

    void loadResource(String path);

}
```



```java
public
```



```java
public
```



```java
public
```



```java
public
```



```java
public
```



```java
public
```



```java
public
```

