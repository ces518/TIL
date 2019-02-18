### Tomcat 7 Issue 

- Tomcat7 의 javax.el-api 의 버전이 낮아서 발생하는 에러 .. 
- pom.xml에 el-api를 3.0으로 의존성을 추가하더라도 
- Tomcat이 Runtime시에 가로채 톰캣의 el-api를 사용해버린다..
- Tomcat Version 8 이상에선 el-api 3.0 이므로 version up 을 해보자..
```
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: Invocation of init method failed; nested exception is javax.persistence.PersistenceException: [PersistenceUnit: default] Unable to build Hibernate SessionFactory; nested exception is org.hibernate.cfg.beanvalidation.IntegrationException: Error activating Bean Validation integration
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1762)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:593)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:515)
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1105)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:867)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:142)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:775)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:397)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:316)
	at org.springframework.boot.web.servlet.support.SpringBootServletInitializer.run(SpringBootServletInitializer.java:157)
	at org.springframework.boot.web.servlet.support.SpringBootServletInitializer.createRootApplicationContext(SpringBootServletInitializer.java:137)
	at org.springframework.boot.web.servlet.support.SpringBootServletInitializer.onStartup(SpringBootServletInitializer.java:91)
	at org.springframework.web.SpringServletContainerInitializer.onStartup(SpringServletContainerInitializer.java:171)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5669)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:145)
	... 43 more
Caused by: javax.persistence.PersistenceException: [PersistenceUnit: default] Unable to build Hibernate SessionFactory; nested exception is org.hibernate.cfg.beanvalidation.IntegrationException: Error activating Bean Validation integration
	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.buildNativeEntityManagerFactory(AbstractEntityManagerFactoryBean.java:402)
	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.afterPropertiesSet(AbstractEntityManagerFactoryBean.java:377)
	at org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean.afterPropertiesSet(LocalContainerEntityManagerFactoryBean.java:341)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1821)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1758)
	... 62 more
Caused by: org.hibernate.cfg.beanvalidation.IntegrationException: Error activating Bean Validation integration
	at org.hibernate.cfg.beanvalidation.BeanValidationIntegrator.integrate(BeanValidationIntegrator.java:137)
	at org.hibernate.internal.SessionFactoryImpl.<init>(SessionFactoryImpl.java:276)
	at org.hibernate.boot.internal.SessionFactoryBuilderImpl.build(SessionFactoryBuilderImpl.java:467)
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl.build(EntityManagerFactoryBuilderImpl.java:939)
	at org.springframework.orm.jpa.vendor.SpringHibernateJpaPersistenceProvider.createContainerEntityManagerFactory(SpringHibernateJpaPersistenceProvider.java:57)
	at org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean.createNativeEntityManagerFactory(LocalContainerEntityManagerFactoryBean.java:365)
	at org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.buildNativeEntityManagerFactory(AbstractEntityManagerFactoryBean.java:390)
	... 66 more
Caused by: java.lang.NoClassDefFoundError: javax/el/ELManager
	at org.hibernate.validator.messageinterpolation.ResourceBundleMessageInterpolator.buildExpressionFactory(ResourceBundleMessageInterpolator.java:88)
	at org.hibernate.validator.messageinterpolation.ResourceBundleMessageInterpolator.<init>(ResourceBundleMessageInterpolator.java:47)
	at org.hibernate.validator.internal.engine.ConfigurationImpl.getDefaultMessageInterpolator(ConfigurationImpl.java:474)
	at org.hibernate.validator.internal.engine.ConfigurationImpl.getDefaultMessageInterpolatorConfiguredWithClassLoader(ConfigurationImpl.java:650)
	at org.hibernate.validator.internal.engine.ConfigurationImpl.getMessageInterpolator(ConfigurationImpl.java:397)
	at org.hibernate.validator.internal.engine.ValidatorFactoryImpl.<init>(ValidatorFactoryImpl.java:183)
	at org.hibernate.validator.HibernateValidator.buildValidatorFactory(HibernateValidator.java:38)
	at org.hibernate.validator.internal.engine.ConfigurationImpl.buildValidatorFactory(ConfigurationImpl.java:364)
	at javax.validation.Validation.buildDefaultValidatorFactory(Validation.java:103)
	at org.hibernate.cfg.beanvalidation.TypeSafeActivator.getValidatorFactory(TypeSafeActivator.java:501)
	at org.hibernate.cfg.beanvalidation.TypeSafeActivator.activate(TypeSafeActivator.java:84)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.hibernate.cfg.beanvalidation.BeanValidationIntegrator.integrate(BeanValidationIntegrator.java:131)
	... 72 more

```