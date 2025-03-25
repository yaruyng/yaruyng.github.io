---
title: "Spring Architecture Series-3.Understanding Spring MVC Implementation"
meta_title: ""
description: ""
date: 2025-03-17T00:00:00Z
image: ""
categories: ["Spring", "Java"]
author: "yaruyng"
tags: ["Spring", "Java"]
draft: false
---
Understanding Spring MVC Implementation
<!--more-->

## Introduction

Spring MVC is a powerful web framework that implements the Model-View-Controller pattern.In this article,I'll explore the implementation of a simplified Spring MVC framework through my miniSpring project,which captures the essential features of Spring MVC while maintaining clarity and simplicity.
```text
src/com/yaruyng/web/
├── servlet/
│   ├── DispatcherServlet.java
│   ├── HandlerMapping.java
│   ├── HandlerAdapter.java
│   ├── ModelAndView.java
│   ├── ViewResolver.java
│   └── View.java
├── method/
├── context/
└── RequestMapping.java
```
## The Front Controller:DispatcherServlet
The **DispatcherServlet** acts as the front controller in our MVC implements:
```java
public class DispatcherServlet extends HttpServlet {
    private WebApplicationContext webApplicationContext;
    private HandlerMapping handlerMapping;
    private HandlerAdapter handlerAdapter;
    private ViewResolver viewResolver;

    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        this.parentApplicationContext = (WebApplicationContext) this.getServletContext()
                .getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
        this.sContextConfigLocation = config.getInitParameter("contextConfigLocation");

        this.webApplicationContext = new AnnotationConfigWebApplicationContext(
            sContextConfigLocation, 
            this.parentApplicationContext
        );
        Refresh();
    }

    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) 
            throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerMethod handlerMethod = null;
        ModelAndView mv = null;
        
        // 1. Get handler for request
        handlerMethod = this.handlerMapping.getHandler(processedRequest);
        if(handlerMethod == null) {
            return;
        }
        
        // 2. Execute handler with adapter
        HandlerAdapter ha = this.handlerAdapter;
        mv = ha.handle(processedRequest, response, handlerMethod);
        
        // 3. Render view
        render(request, response, mv);
    }
}
```
Key features of the Dispatcher:
1. Initializes the web application context
2. Sets up handler mapping,adapter, and view resolver
3. Processes incoming requests through the **doDispatch** method
4. Coordinates between different components of the MVC framework

## Request Mapping
The **@RequestMapping** annotation is used to map web requests to handler methods:
```java
@Target(value = {ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestMapping {
    String value() default "";
}
```
This simple yet powerful annotation allows for:
- Method-level request mapping
- URL pattern matching
- Runtime processing of mapping

## Model and View Handling
The **ModelAndView** class represents both the model data and view information:
```java
public class ModelAndView {
    private Object view;
    private Map<String, Object> model = new HashMap<>();
    
    public ModelAndView(String viewName, Map<String, ?> modelData) {
        this.view = viewName;
        if(modelData != null) {
            addAllAttributes(modelData);
        }
    }
    
    public void addAttribute(String attributeName, Object attributeValue) {
        model.put(attributeName, attributeValue);
    }
    
    public ModelAndView addObject(String attributeName, Object attributeValue) {
        addAttribute(attributeName, attributeValue);
        return this;
    }
    
    // Other methods...
}
```
This implementation:
1. Combines view information with model data
2. Provides flexible constructors fro different use cases
3. Offers convenient methods for adding attributes

## View Resolution
The view resolution process is handled the **ViewResolver** interface and its implementation:
```java
public interface ViewResolver {
    View resolveViewName(String viewName) throws Exception;
}
```
The actual rendering is performed by the **View** interface:
```java
public interface View {
    void render(Map<String, Object> model, HttpServletRequest request,
                HttpServletResponse response) throws Exception;
}
```
## Request Processing Flow
The request processing flow in miniSpring MVC follows these steps:
1. Request Reception
```java
protected void service(HttpServletRequest request, HttpServletResponse response) {
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.webApplicationContext);
    try {
        doDispatch(request, response);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```
2. Handler Mapping
```java
handlerMethod = this.handlerMapping.getHandler(processedRequest);
```
3. Handler Execution
```java
mv = ha.handle(processedRequest, response, handlerMethod);
```
4. View Rendering
```java
protected void render(HttpServletRequest request, HttpServletResponse response, 
        ModelAndView mv) throws Exception {
    if(mv == null) {
        response.getWriter().flush();
        response.getWriter().close();
        return;
    }

    String sTarget = mv.getViewName();
    Map<String, Object> modelMap = mv.getModel();
    View view = resolveViewName(sTarget, modelMap, request);
    view.render(modelMap, request, response);
}
```
## Integration with IoC Container
The MVC framework integrates with the IoC container through the **WebApplicationContext**:
```java
protected void initHandlerMappings(WebApplicationContext wac) {
    try {
        this.handlerMapping = (HandlerMapping) wac.getBean(HANDLER_MAPPING_BEAN_NAME);
    } catch (BeansException e) {
        e.printStackTrace();
    }
}
```
This integration provides:
- Automatic bean management
- Dependency injection for controllers
- Configuration management

## Example Usage
Here's how to use the miniSpring MVC framework:
```java
@RequestMapping("/hello")
public ModelAndView handleRequest(HttpServletRequest request) {
    String message = "Hello, World!";
    ModelAndView mv = new ModelAndView("hello");
    mv.addObject("message", message);
    return mv;
}
```
And the corresponding configuration:
```xml
<bean class="com.yaruyng.web.servlet.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/hello">helloController</prop>
        </props>
    </property>
</bean>
```
## Performance Considerations
The implementation includes several performance optimizations:
1. Single Front Controller: All request go through a single servlet
2. Handler Caching: Handler methods are cached after first resolution
3. View Caching: Resolved views are cached for reuse
4. Efficient Request Processing: Minimal object creation per request

## Extension Points
The framework provides several extension points:
1. Custom Handler Mappings: Implement HandlerMapping interface
2. Custom View Resolvers: Implement ViewResolver interface
3. Custom Handler Adapters: Implement HandlerAdapter interface
4. Custom Views: Implement View interface

## Conclusion
This implementation of Spring MVC demonstrates:
1. The power of the front controller pattern
2. Clean separation of concerns through MVC
3. Flexible handling of requests and responses
4. Integration with IoC container
5. Extensible architecture
Key takeaways:
1. Understanding Spring MVC's internal structure
2. How different components work together
3. The importance of clean architecture
4. Extension points for customization
5. 
The miniSpring MVC implementation provides a solid foundation for understanding how Spring MVC works internally while maintaining simplicity and clarity.
