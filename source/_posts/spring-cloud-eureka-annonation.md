---
layout:     post
title:      Spring Cloud中@EnableEurekaClient注解的源码分析
date:       2016-11-05 14:00:00 +0800
summary:    Spring Cloud Eureka服务续约和服务下线源码分析
toc: true
categories:
- Spring Cloud Eureka
tags:
- Spring Cloud Eureka
---
**摘要**:在上一篇中，主要分析了Eureka 服务注册的源码分析，在本篇文章中将对Eureka的Renew(服务续约)，cancel(服务下线)进行源码分析。
## Eureka client简介
Eureka client 负责与Eureka Server 配合向外提供注册与发现服务接口。首先看下eureka client是怎么定义，Netflix的 eureka client的行为在LookupService中定义，Lookup service for finding active instances，定义了，从outline中能看到起“规定”了如下几个最基本的方法。
服务发现必须实现的基本行为：
com.netflix.discovery.shared.LookupService<T>

com.netflix.discovery.DiscoveryClient是netflix使用的客户端，从其class的注释可以看到他主要做这几件事情：
a) Registering the instance with Eureka Server
b) Renewalof the lease with Eureka Server
c) Cancellation of the lease from Eureka Server during shutdown
com.netflix.discovery.DiscoveryClient

org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient


##@EnableEurekaClient注解
### @EnableEurekaClient使用
1. 用过spring cloud的同学都知道，如下代码
```java
@EnableEurekaClient
@SpringBootApplication
public class CloudEurekaClientApplication {

	public static void main(String[] args) {
		new SpringApplicationBuilder(CloudEurekaClientApplication.class).web(true).run(args);
	}

}
```
通过@EnableEurekaClient这个简单的注解，在spring cloud应用启动的时候，就可以把EurekaDiscoveryClient注入，继而使用NetFlix提供的Eureka client。

2. 打开EnableEurekaClient这个类，可以看到这个自定义的annotation @EnableEurekaClient里面没有内容。只是个属于那种标记类型 的注释，他的作用就是开启Eureka discovery的配置，正是通过这个标记，autoconfiguration就可以加载相关的Eureka类。那我们看下他是怎么做到的。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@EnableDiscoveryClient
public @interface EnableEurekaClient {

}
```
在上述代码中，我们看到，EnableEurekaClient上面加入了另外一个注解@EnableDiscoveryClient，看看这个注解的代码如下所示:
```java
/**
 * Annotation to enable a DiscoveryClient implementation.
 * @author Spencer Gibb
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(EnableDiscoveryClientImportSelector.class)
public @interface EnableDiscoveryClient {

}
```
这个注解import了EnableDiscoveryClientImportSelector.class这样一个类，其实就是通过这个类来加载需求要用到的bean。
点开EnableDiscoveryClientImportSelector类，如下代码:
```java
@Order(Ordered.LOWEST_PRECEDENCE - 100)
public class EnableDiscoveryClientImportSelector
		extends SpringFactoryImportSelector<EnableDiscoveryClient> {

	@Override
	protected boolean isEnabled() {
		return new RelaxedPropertyResolver(getEnvironment()).getProperty(
				"spring.cloud.discovery.enabled", Boolean.class, Boolean.TRUE);
	}

	@Override
	protected boolean hasDefaultFactory() {
		return true;
	}

}
```
看到这里有覆盖了父类SpringFactoryImportSelector的一个方法isEnabled，其实标记体现的也就是这里，在配置的环境中找这个key是否被配置，注意，默认是TRUE，也就是只要import了这个配置，就会enable。

在其父类org.springframework.cloud.commons.util.SpringFactoryImportSelector
的String[] selectImports(AnnotationMetadata metadata)方法中正是根据这个标记类判定是否加载如下定义的类。在第59行，代码如下所示。
```java
@Override
	public String[] selectImports(AnnotationMetadata metadata) {
		if (!isEnabled()) {
			return new String[0];
		}
		AnnotationAttributes attributes = AnnotationAttributes.fromMap(
				metadata.getAnnotationAttributes(this.annotationClass.getName(), true));

		Assert.notNull(attributes, "No " + getSimpleName() + " attributes found. Is "
				+ metadata.getClassName() + " annotated with @" + getSimpleName() + "?");

		// Find all possible auto configuration classes, filtering duplicates
		List<String> factories = new ArrayList<>(new LinkedHashSet<>(SpringFactoriesLoader
				.loadFactoryNames(this.annotationClass, this.beanClassLoader)));

		if (factories.isEmpty() && !hasDefaultFactory()) {
			throw new IllegalStateException("Annotation @" + getSimpleName()
					+ " found, but there are no implementations. Did you forget to include a starter?");
		}

		if (factories.size() > 1) {
			// there should only ever be one DiscoveryClient, but there might be more than
			// one factory
			log.warn("More than one implementation " + "of @" + getSimpleName()
					+ " (now relying on @Conditionals to pick one): " + factories);
		}

		return factories.toArray(new String[factories.size()]);
	}

```
在源码中70-71行，即在
org.springframework.core.io.support.SpringFactoriesLoader 中的109行的loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader)方法
```java
  public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		try {
			Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			List<String> result = new ArrayList<String>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
				String factoryClassNames = properties.getProperty(factoryClassName);
				result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
			}
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() +
					"] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```
实际调用loadFactoryNames其实加载META-INF/spring.factories下的class。
```java
  /**
	* The location to look for factories.
	* <p>Can be present in multiple JAR files.
  */
  public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```
而在spring-cloud-netflix-eureka-client\src\main\resources\META-INF\spring.factories中配置，
![spring.factories](/images/spring-cloud-netflix/eureka/anoation/1.png)
可以看到EnableAutoConfiguration的包含了EurekaClientConfigServerAutoConfiguration。
```factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.netflix.eureka.config.EurekaClientConfigServerAutoConfiguration,\
org.springframework.cloud.netflix.eureka.config.EurekaDiscoveryClientConfigServiceAutoConfiguration,\
org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration,\
org.springframework.cloud.netflix.ribbon.eureka.RibbonEurekaAutoConfiguration

org.springframework.cloud.bootstrap.BootstrapConfiguration=\
org.springframework.cloud.netflix.eureka.config.EurekaDiscoveryClientConfigServiceBootstrapConfiguration

org.springframework.cloud.client.discovery.EnableDiscoveryClient=\
org.springframework.cloud.netflix.eureka.EurekaDiscoveryClientConfiguration

```
打开org.springframework.cloud.netflix.eureka.config.EurekaClientConfigServerAutoConfiguration可以看到
EurekaClientAutoConfiguration分别会依次注入配置EurekaClientConfigBean、 EurekaInstanceConfigBean和DiscoveryClient