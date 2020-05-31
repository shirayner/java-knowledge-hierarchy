# Spring容器的基本实现

[toc]

## 前言

### 推荐阅读

> - [Spring BeanFactory 类图详解](https://blog.csdn.net/qq_34090008/article/details/78772189)
> - [spring 主要涉及类类图](https://blog.csdn.net/strivezxq/article/details/44560771)
> - [Spring容器](https://www.jianshu.com/p/ab1bf09f6501)

## 一、基本概念

### 1.容器相关类图



#### 1.1 容器概览

```plantuml
@startuml

interface ApplicationContext #Orange extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

}

/'
' 1.BeanFactory
'/
interface BeanFactory #Orange {
}

interface AutowireCapableBeanFactory extends BeanFactory {

}

interface ListableBeanFactory extends BeanFactory {

}

interface HierarchicalBeanFactory extends BeanFactory {

}


interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

}


interface AliasRegistry {

}

class SimpleAliasRegistry implements AliasRegistry {

}

class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

}

abstract class FactoryBeanRegistrySupport extends DefaultSingletonBeanRegistry {

}

abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {

}

abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {

}

interface ConfigurableListableBeanFactory extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {

}

class DefaultListableBeanFactory #LightSalmon extends AbstractAutowireCapableBeanFactory implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {

}


class XmlBeanFactory #Cyan extends DefaultListableBeanFactory {

}


/'
' 2.ApplicationEventPublisher、EnvironmentCapable、MessageSource
'/
interface ApplicationEventPublisher {

}

interface EnvironmentCapable {

}

interface MessageSource {

}


/'
' 3.ResourceLoader
'/
interface ResourceLoader {

}

interface ResourcePatternResolver extends ResourceLoader {

}

class DefaultResourceLoader implements ResourceLoader {

}


'note left of ApplicationContext: On last defined class

/'
' 4.ApplicationContext
'/
interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {

}

interface Lifecycle {

}

interface Closeable extends AutoCloseable {

}

interface AutoCloseable {

}

abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {

}

abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {

}

abstract class AbstractRefreshableConfigApplicationContext extends AbstractRefreshableApplicationContext implements BeanNameAware, InitializingBean {


}

interface InitializingBean {

}

interface BeanNameAware extends Aware {

}

interface Aware {

}

/'
' 4.1 AbstractXmlApplicationContext
'/
abstract class AbstractXmlApplicationContext #LightSalmon extends AbstractRefreshableConfigApplicationContext {

}

class ClassPathXmlApplicationContext #Cyan extends AbstractXmlApplicationContext {

}

class FileSystemXmlApplicationContext #Cyan extends AbstractXmlApplicationContext {

}


/'
' 4.2 AbstractRefreshableWebApplicationContext
'/

abstract class AbstractRefreshableWebApplicationContext #LightSalmon extends AbstractRefreshableConfigApplicationContext implements ConfigurableWebApplicationContext, ThemeSource {

}

interface ThemeSource {

}

class XmlWebApplicationContext #Cyan extends AbstractRefreshableWebApplicationContext {

}

interface ConfigurableWebApplicationContext extends WebApplicationContext, ConfigurableApplicationContext {

}

interface WebApplicationContext extends ApplicationContext {

}


class GenericApplicationContext #LightSalmon extends AbstractApplicationContext implements BeanDefinitionRegistry {

}

interface BeanDefinitionRegistry extends AliasRegistry {

}

interface AliasRegistry {

}

class AnnotationConfigApplicationContext #Cyan extends GenericApplicationContext implements AnnotationConfigRegistry {

}

interface AnnotationConfigRegistry {

}


class AnnotationConfigWebApplicationContext #Cyan extends AbstractRefreshableWebApplicationContext implements AnnotationConfigRegistry {

}

@enduml
```

#### 1.2 DefaultListableBeanFactory

```plantuml
@startuml
interface BeanFactory #Orange {
}

interface AutowireCapableBeanFactory extends BeanFactory {

}

interface HierarchicalBeanFactory extends BeanFactory {

}

interface ListableBeanFactory extends BeanFactory {

}


interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

}

interface AliasRegistry {

}

class SimpleAliasRegistry implements AliasRegistry {

}

class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

}

abstract class FactoryBeanRegistrySupport extends DefaultSingletonBeanRegistry {

}

abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {

}

abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {

}

interface ConfigurableListableBeanFactory extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {

}

class DefaultListableBeanFactory #LightSalmon extends AbstractAutowireCapableBeanFactory implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {

}

interface BeanDefinitionRegistry extends AliasRegistry {

}

class XmlBeanFactory extends DefaultListableBeanFactory {

}



@enduml
```







