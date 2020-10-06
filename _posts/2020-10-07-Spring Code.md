---
layout: post
title: Spring Code
tags: Spring
---
## 容器实现
```
/** @deprecated */
BeanFactory bf = new XmlBeanFactory(new ClassPathResource("beanFactoryTest.xml"));
```

- ClassPathResource继承自**Resource类**，类似的还有FileSystemResource、UrlResrouce、InputStreamResource、ByteArrayResource。Resource类实现了InputStreamSource接口，可以调用getInputStream直接获取InputStream
- XmlBeanFactory构造方法，调用父类AbstractAutowireCapableBeanFactory忽略给定接口的自动装配功能，例如A中的属性B，B实现了BeanNameAware接口，则会被忽略；loadBeanDefinitions
- XmlBeanDefinitionReader的**loadBeanDefinitions**，输入流编码，doLoadBeanDefinitions。doLoadBeanDefinitions校验XML格式，加载XML文件得到Document，返回Document注册Bean的信息
- DefaultDocumentLoader的loadDocument，创建factory，通过factory创建builder，解析inputSource返回**Document对象**
- DefaultBeanDefinitionDocumentReader的doRegisterBeanDefinitions，传入参数为root = doc.getDocumentElement()，首先判断profile（开发环境），后在**模板方法模式**中parseBeanDefinitions方法中解析注册

```
<beans profile="dev">
<context-param>
  <param-name>Spring.profiles.active</param-name>
  <param-value>dev<param-value/>
</context-param>

this.preProcessXml(root);//空方法，解析前处理，留给子类实现，模板方法模式
this.parseBeanDefinitions(root, this.delegate);
this.postProcessXml(root);//空方法，解析后处理，留给子类实现，模板方法模式
```

- DefaultBeanDefinitionDocumentReader的parseBeanDefinitions，**两大类Beans的声明**，默认的`<bean id="test" class="test.TestBean">`和自定义的`<tx:annotation-driven>`

```
Element ele = (Element)node; //根节点或子节点
if (delegate.isDefaultNamespace(ele)) { //是否是默认命名空间http://www.Springframework.org/schema/beans
        this.parseDefaultElement(ele, delegate);
    } else {
        delegate.parseCustomElement(ele);
}
```

### 默认标签解析parseDefaultElement
1. bean即processBeanDefinition
- BeanDefinitionHolder类的实例bdHolder，其中包含了配置文件的各个属性
    - BeanDefinitionParserDelegate的parseBeanDefinitionElement，解析id和name，解析其他属性parseBeanDefinitionElement，beanName(id)为空时生成beanName，返回BeanDefinitionHolder
    - BeanDefinitionParserDelegate的parseBeanDefinitionElement，parseBeanDefinitionAttributes对element所有元素解析，后续解析元数据（meta）、lookup-method（用于动态替换抽象方法返回实体类）、replaced-method（用于动态替换方法实现，需要新类实现MethodReplacer方法）、constructor-arg、property、qualifier（指定注入的bean名称），返回AbstractBeanDefinition的子类GenericBeanDefinition
- 默认标签的子节点下存在自定义属性，解析自定义标签
    - `<bean id=...> <mybean user username="aa"/> </bean>`不是以bean形式存在的，而是自定义属性
    - decorateIfRequired获取自定义标签的命名空间，自定义类型所对应的NamespaceHandler
- bdHolder注册
    - DefaultListableBeanFactory类registerBeanDefinition，校验methodOverrides是否与工厂方法并存或对应方法不存在，校验beanName已经注册且配置了不允许被覆盖，存入ConcurrentHashMap(256).put(beanName, beanDefinition) ，清除解析前缓存
    - SimpleAliasRegistry类registerAlias，beanName和alias相同删除原有key为alias的项结束，存在key为alias的value为beanName结束，循环检查，注册ConcurrentHashMap(16).put(alias, name);
```
方式一name为别名
<bean id="testBean"  name="testBean,testBean2"  class=“com.test"/>
方式二
<bean id="testBean"  class="com.test"/>
<alias name="testBean" alias="testBean,testBean2"/>
```

- 通知监听注册BeanDefinition事件的监听器，Spring并未做逻辑

2. alias即processAliasRegistration
同上文SimpleAliasRegistry类registerAlias

3. import即importBeanDefinitionResource
- 获取resource属性表示路径
- 解析路径中的系统属性，例如`${user.dir}`
- 绝对路径加载配置文件
- 相对路径计算绝对路径加载配置文件
- 通知监听器

4. bean即doRegisterBeanDefinitions
回调doRegisterBeanDefinitions

### 自定义标签解析parseCustomElement
1. 自定义标签使用
- 创建POJO
- 创建XSD
```
//tns为命名空间
<schema xmlns="..." targetNamespace="http://www.test.com/schema/user" xmlns:tns="http://www.test.com/schema/user" elementFormDefault="qualified"/>
  <element name="user">
    <complexType>
      <attribute name="id" type="string"/>
      <attribute name="name" type="string"/>
      <attribute name="email" type="string"/>
    </complexType>
  </element>
</schema>
```

- 创建类A实现BeanDefinitionParser接口，例如继承AbstractSingleBeanDefinitionParser类，`getBeanClass`方法返回`User.class`，`doParse`方法中`bean.addPropertyValue("userName", element.getAttribute("userName"));`
- 创建类B继承NamespaceHandlerSupport，组件注册到Spring，重写init方法为`registerBeanDefinitionParser("user",new A())`即自定义标签`<user:xxx`这类user开头的交给A类处理
- 编写Spring.handlers`http://www.test.com/schema/user=test.B`
- 编写Spring.schemas`http://www.test.com/schema/user=META-INF/test.xsd`
- 使用

```
<beans ... xmlns:myname="http://www.test.com/schema/user" xsi:... http://www.test.com/schema/user http://www.test.com/schema/user.xsd />
  <myname:user id="" userName="" email=""/>
</beans>
```
2. 解析
BeanDefinitionParserDelegate类parseCustomElement，获取命名空间，获取对应NamespaceHandler，调用handler解析

- DefaultNamespaceHandlerResolver类resolve，将META-INT/Spring.handlers添加并获取ConcurrentHashMap(String namespaceUrl, Object handler)，
    - handler为空返回null
    - 已经解析(instanceof NamespaceHandler)过直接返回
    - 未解析则反射将类路径转化为类，初始化类，调用init方法，put
- 调用handler解析
    - NamespaceHandlerSupport类findParserForElement，获取元素名，例如`<myname:user`中的`user`，找对应解释器
    - AbstractBeanDefinitionParser类parse，调用AbstractSingleBeanDefinitionParser类parseInternal返回解析后的AbstractBeanDefinition，转化为BeanDefinitionHolder并注册，获取自定义
    - parseInternal，调用`getBeanClass`方法返回class，使用父类的scope属性，配置延迟加载，调用子类的`doParse`方法

## 加载bean
```
MyTestBean bean = (MyTestBean) bf.getBean("myTestBean");
```
AbstractBeanFactory类doGetBean

- 别名map查找beanName
- 缓存中获取单例bean，存在则bean的实例化，类型转换，**返回**
- 原型模式检查
- 检测parentBeanFactory，xml配置文件类GenericBeanDefinition转为RootBeanDefinition，递归加载依赖bean
- 创建bean，bean的实例化，类型转换，**返回**

### FactoryBean的使用
一般Spring通过反射利用bean的class属性指定实现类实例化bean，bean中大量配置信息时，灵活受限。Spring提供了FactoryBean工厂类接口，实现接口定制实例化bean逻辑
```
//工厂版
public class CarFactoryBean implements FactoryBean<Car> {
    private String carInfo;
    //getter and setter
    public Car getObject() throws Exception {
        Car car = new Car();
        String[] infos = carInfo.split(",");
        ...
        return car;
    }
    public Class<Car> getObjectType() {
        return Car.class;
    }
    public boolean isSingleton() {
        return false;
    }
}

<bean id="car" class="...CarFactoryBean" carInfo="超跑,400,200">
//普通版
<bean id="car" class="...Car">
  <property name="type" value="超跑"/>
</bean>
```
`bf.getBean("car")`，反射机制发现CarFactoryBean实现了FactoryBean接口，调用CarFactoryBean#getObject()。如果需要返回CarFactoryBean实例，则`bf.getBean("&car")`

### 循环依赖
- 构造器循环依赖，无法解决。
- settter循环依赖，单例可以解决。创建ABean，暴漏一个ObjectFactory，返回一个提前暴露一个创建中的bean，A标识符放入当前bean池
- prototype循环依赖，无法解决。


### 缓存中获取单例bean
DefaultSingletonBeanRegistry类getSingleton，返回`Object sharedInstance = singletonObject`

- singletonObjects保存BeanName和bean实例，bean实例获取singletonFactory.getObject()
- earlySingletonObjects保存BeanName和bean实例，bean还在创建中，getBean方法可以获取，用来检测循环利用
- singletonFactory保存BeanName和bean工厂
- registeredSingletons保存已注册bean

```
if (singletonFactory != null) {
    singletonObject = singletonFactory.getObject();
    this.earlySingletonObjects.put(beanName, singletonObject);
    this.singletonFactories.remove(beanName);
}
```

### 准备Bean
AbstractAutowireCapableBeanFactory的createBean

- 对override属性标记验证，`lookup-method`和`replace-method`，AOP
- resolveBeforeInstantiation初始化前的后处理器，解析是否初始化前短路
```
// 实例化前后处理器
bean = this.applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
if (bean != null) {
    // 实例化后后处理器
    bean = this.applyBeanPostProcessorsAfterInitialization(bean, beanName);
}
```

- doCreateBean创建bean
    - 单例则清除缓存
    - 实例化bean，BeanDefinition转换为BeanWrapper
    - bean合并后的处理，Autowired
    - 循环依赖处理
    - 属性填充
    - 注册DisposableBean，`destroy-method`

### 创建Bean
doCreateBean

- 实例化bean，createBeanInstance
    - RootBeanDefinition存在FactoryMethodName，即`factory-method`，instantiateUsingFactoryMethod生成bean实例
    - RootBeanDefinition存在resolvedConstructorOrFactoryMethod缓存，否则需要再次解析
    - RootBeanDefinition中constructorArgumentsResolved，true构造函数自动注入autowireConstructor，false默认构造函数instantiateBean（直接实例化Bean）
        - 参数确定：explicitArgs使用getBean(String name, Object... args)的参数选择调用不同的构造函数，缓存中参数选择构造函数，配置文件获取getConstructorArgumentValues构造函数信息
        - 构造函数确定：分别参数数量降序排序public构造方法和非public构造方法，分为有参和无参确定构造函数
        - 实例化Bean：SimpleInstantiationStrategy的instantiate，首先判断是否使用replace和lookup配置方法，未使用反射创建，否则CglibSubclassingInstantiationStrategy类instantiateWithMethodInjection返回包含拦截器的动态代理
- 记录bean的ObjectFactory，解决循环依赖
    - earlySingletonExposure代表是否单例、是否允许循环、是否对应的bean正在创建
    - 均是则addSingletonFactory方法，其传入参数ObjectFactory使用getEarlyBeanReference产生，方法中AOP将advice动态**织入**
- 属性注入，populateBean 
    - InstantiationAwareBeanPostProcessor处理器的postProcessAfterInstantiation
    - 根据注入类型byName和byType提取依赖bean，存入PropertyValues
    - InstantiationAwareBeanPostProcessor处理器的postProcessPropertyValues
    - 填充PropertyValues至BeanWrapper
- 初始化bean，initializeBean
    - 激活Aware方法，invokeAwareMethods，Aware接口作用例如实现BeanFactoryAware的bean初始化后，Spring容器将注入BeanFactory实例
    - applyBeanPostProcessorsBeforeInitialization，BeanPostProcessor的postProcessBeforeInitialization
    - invokeInitMethods，**先**自定义bean实现InitializingBean接口，afterPropertiesSet中实现初始化，**后init-method**
    - applyBeanPostProcessorsAfterInitialization，BeanPostProcessor的postProcessAfterInitialization
- 注册DisposableBean，**destroy-method**和注册后处理器**DestructionAwareBeanPostProcessor**
### 获取单例
上面是缓存中获取单例，当不存在时，从头开始加载bean，DefaultSingletonBeanRegistry的getSingleton，包含传入参数`ObjectFactory<?> singletonFactory`

- 为未加载`singletonObject == null`
- 记录正在加载`singletonsCurrentlyInCreation.add(beanName)`
- 调用传入参数的方法`singletonFactory.getObject()`
- 移除正在加载`singletonsCurrentlyInCreation.remove(beanName)`
- 结果记录至缓存
```
synchronized(this.singletonObjects) {
    this.singletonObjects.put(beanName, singletonObject);
    this.singletonFactories.remove(beanName);
    this.earlySingletonObjects.remove(beanName);
    this.registeredSingletons.add(beanName);
}
```

### bean实例化
从`sharedInstance,prototypeInstance,scopedInstance`获取对象

- AbstractBeanFactory类getObjectForBeanInstance，非FactoryBean不做处理，FactoryBean检验开头&符号，**返回**，XML配置文件Genericbeandefinition转RootBeanDefinition
- 父类FactoryBeanRegistrySupport的doGetObjectFromFactoryBean，`object = factory.getObject()`
