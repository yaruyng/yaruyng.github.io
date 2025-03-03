---
title: "Spring Architecture Series-1.Understanding the Principles of IoC Container Implementation"
meta_title: ""
description: ""
date: 2025-03-03T00:00:00Z
image: ""
categories: ["Spring", "Java"]
author: "yaruyng"
tags: ["Spring", "Java"]
draft: false
---
Understanding the Principles of IoC Container Implementation
<!--more-->

## Introduction

The Inversion of Control (IoC) container is the cornerstone of the Spring Framework.It manages object creation, configuration, and lifecycle, allowing developers to focus on business logic rather than infrastructure concerns.In this article, I'll dive deep into the principles behind IoC container implementation by examing my miniSpring project - a simplified version of the Spring Framework I built to understand its inner workings.

## What is Inversion of Control?

Inversion of Control is a design principle where the control flow of a program is inverted: instead of the programmer controlling the flow, the framework takes charge.In the context of dependency management, IoC means that objects do not create others objects from an external source - the IoC container.

## Core Components of IoC Container

### The BeanFactory Interface

At the Heart of Spring's IoC container is the **BeanFactory**

```java
public interface BeanFactory {
    Object getBean(String beanName) throws BeansException;
    boolean containsBean(String name);
    boolean isSingleton(String name);
    boolean isPrototype(String name);
    Class<?> getType(String name);
}
```

This interface provides methods to retrieve beans, check their existence,
determine their scope (single or prototype), and get their type information.

### BeanDefinition: The Blueprint fro Beans

Before a bean can be created, the container needs to konw how to construct it. This information is encapsulated in a **BeanDefinition** object, which serves as a blueprint for creating beans:

```java
public class BeanDefinition {
    private String id;
    private String className;
    private String scope = SCOPE_SINGLETON;
    private boolean lazyInit = false;
    private String[] dependsOn;
    private PropertyValues propertyValues;
    private ConstructorArgumentValues constructorArgumentValues;
    private String initMethodName;
    // Getters and setters
}
```

### The Bean Creation Process

The bean creation process in my miniSpring implementation flolows these steps:

1. **Loading Bean Definitions**: The container reads bean definitions from configuration sources(XML, annotations, etc)
2. **Bean Instantiation**: The container create bean instances based on their definitions
3. **Dependency injection**: The container injects dependencies into beans
4. **Bean Post-Processing**: The container applies post-processors to modify beans
5. **Initialization**: The container calls initialization methods on beans
6. **Ready fro Use**:  The fully configured beans are ready for use
Let's examine each step in detal.

## Loading Bean Definitions

In miniSpring I implemented on XML-based configuration approach similar to traditional Spring:

```java
public class XmlBeanDefinitionReader {
    AbstractBeanFactory bf;
    
    public XmlBeanDefinitionReader(AbstractBeanFactory bf) {
        this.bf = bf;
    }
    
    public void loadBeanDefinitions(Resource resource) {
        while (resource.hasNext()) {
            Element element = (Element) resource.next();
            String beanID = element.attributeValue("id");
            String beanClassName = element.attributeValue("class");
            BeanDefinition beanDefinition = new BeanDefinition(beanID, beanClassName);
            
            // Parse property elements
            List<Element> propertyElements = element.elements("property");
            PropertyValues pvs = new PropertyValues();
            List<String> refs = new ArrayList<>();
            
            for (Element e: propertyElements) {
                // Parse property attributes
                // ...
                pvs.addPropertyValue(new PropertyValue(pType, pName, pV, isRef));
            }
            
            beanDefinition.setPropertyValues(pvs);
            beanDefinition.setDependsOn(refArray);
            
            // Register the bean definition
            bf.registerBeanDefinition(beanID, beanDefinition);
        }
    }
}
```

The **XmlBeanDefinitionReader** parses XML configuration files, extracts bean definitions, and registers them with the bean factory.

## Bean Instantiation and Dependency Injection

The **AbstractBeanFactory** class handles bean instantiation and dependency injection:

```java
public Object getBean(String beanName) throws BeansException {
    Object singleton = this.getSingleton(beanName);

    if (singleton == null) {
        singleton = this.earlySingletonObjects.get(beanName);
        if (singleton == null) {
            BeanDefinition bd = beanDefinitionMap.get(beanName);
            if (bd != null) {
                singleton = createBean(bd);
                this.registerBean(beanName, singleton);
            }
        }
    }
    
    return singleton;
}

private Object createBean(BeanDefinition bd) {
    Class<?> clz = null;
    Object obj = doCreateBean(bd);
    this.earlySingletonObjects.put(bd.getId(), obj);
    
    try {
        clz = Class.forName(bd.getClassName());
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
    
    populateBean(bd, clz, obj);
    
    return obj;
}
```

The **getBean** method first checks if the requested bean already exists in the singleton cache.If not, it creates a new instance using the **createBean** method, which:

1. Creates the bean instance
2. Adds it to the early singleton cache to handle circular dependencies
3. Populates the bean with its dependencies

## Handing Dependencies

Denpendencybinjection is performed in the populateBean method:
 
 ```java
private void populateBean(BeanDefinition bd, Class<?> clz, Object obj) {
    handleProperties(bd, clz, obj);
}

private void handleProperties(BeanDefinition bd, Class<?> clz, Object obj) {
    // Get property values from bean definition
    PropertyValues propertyValues = bd.getPropertyValues();
    if (propertyValues == null || propertyValues.isEmpty()) {
        return;
    }
    
    for (PropertyValue propertyValue : propertyValues.getPropertyValueList()) {
        String pName = propertyValue.getName();
        String pType = propertyValue.getType();
        Object pValue = propertyValue.getValue();
        boolean isRef = propertyValue.getIsRef();
        
        Class<?>[] paramTypes = new Class<?>[1];
        Object[] paramValues = new Object[1];
        
        if (!isRef) {
            // Handle primitive types
            // ...
        } else {
            // Handle reference types
            try {
                paramTypes[0] = Class.forName(pType);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
            try {
                paramValues[0] = getBean((String) pValue);
            } catch (BeansException e) {
                e.printStackTrace();
            }
        }
        
        // Use reflection to set property values
        String methodName = "set" + pName.substring(0, 1).toUpperCase() + pName.substring(1);
        Method method = null;
        try {
            method = clz.getMethod(methodName, paramTypes);
            method.invoke(obj, paramValues);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
 ```
 
This method:
1. Retrieves property values from the bean definition
2. For each property, determines if it's a value or a reference
3. For reference, recursively calls **getBean** to get the dependency
4. Uses reflection to call the appropriate setter method

## Bean Lifecycle Management

The Ioc container also manages bean lifecycle event:

```java
private void invokeInitMethod(BeanDefinition bd, Object obj) {
    if (bd.getInitMethodName() == null || "".equals(bd.getInitMethodName())) {
        return;
    }
    
    Method method = null;
    try {
        method = obj.getClass().getMethod(bd.getInitMethodName());
        method.invoke(obj);
    } catch (NoSuchMethodException | SecurityException | IllegalAccessException | 
             IllegalArgumentException | InvocationTargetException e) {
        e.printStackTrace();
    }
}
```

The method calls the initialization method specified in the bean definition after the bean fully configured.

## The ApplicationContext

While **BeanFactory** provides basic functionality, Spring's ApplicationContext offers more advanced features. In miniSpring, I implemented ClassPathXmlApplicationContext:

```java
public class ClassPathXmlApplicationContext extends AbstractApplicationContext {
    DefaultListableBeanFactory beanFactory;
    
    public ClassPathXmlApplicationContext(String fileName) {
        Resource res = new ClassPathXmlResource(fileName);
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        reader.loadBeanDefinitions(res);
        this.beanFactory = beanFactory;
        
        try {
            refresh();
        } catch (BeansException e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void refresh() {
        // Initialize the container
        registerBeanPostProcessors(beanFactory);
        onRefresh();
        finishRefresh();
    }
}
```

The **ClassPathXmlApplicationContext** loads bean definitions from an XML file, creates a bean factory, and refreshes the context, which includes:
Registering bean post-processors
Refreshing the bean factory
Publishing a context refresh event

## Conclusion

Implementing an IoC container from scratch has given me a deep understanding of Spring's internal workings. The key components - BeanFactory, BeanDefinition, and the bean lifecycle management - work together to provide a powerful dependency injection framework.
By inverting control of object creation and configuration, Spring allows developers to focus on business logic rather than infrastructure concerns. The IoC container handles the complex tasks of object instantiation, dependency resolution, and lifecycle management, resulting in more modular, testable, and maintainable code.
In future articles, I'll explore other aspects of my miniSpring implementation, including AOP, transaction management, and web MVC. Stay tuned!
