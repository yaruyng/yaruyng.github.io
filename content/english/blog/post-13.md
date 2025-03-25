---
title: "Spring Architecture Series-5.Implementing Mybatis Integration"
meta_title: ""
description: ""
date: 2025-03-24T00:00:00Z
image: ""
categories: ["Spring", "Java"]
author: "yaruyng"
tags: ["Spring", "Java"]
draft: false
---
Implementing Mybatis Integration
<!--more-->

## Introduction
MyBatis is a popular persistence framework that provides a flexible way to map SQL statements to Java objects.In this article, I'll explore how to implements MyBatis integration in aSpring-like framework,base on my miniSpring project's implementation.

## Core Components
The MyBatis integration consists of several key components:
```text
src/com/yaruyng/batis/
├── DefaultSqlSessionFactory.java
├── DefaultSqlSession.java
├── SqlSessionFactory.java
├── SqlSession.java
└── MapperNode.java
```

### SqlSessionFactory Implementation

The **SqlSessionFactory** is the entry point for creating **SqlSession** instance:
```java
public class DefaultSqlSessionFactory implements SqlSessionFactory {
    @Autowired
    JdbcTemplate jdbcTemplate;
    
    String mapperLocations;
    Map<String, MapperNode> mapperNodeMap = new HashMap<>();

    public void init() {
        scanLocation(this.mapperLocations);
        // Initialize mapper nodes
    }

    @Override
    public SqlSession openSession() {
        SqlSession newSqlSession = new DefaultSqlSession();
        newSqlSession.setJdbcTemplate(jdbcTemplate);
        newSqlSession.setSqlSessionFactory(this);
        return newSqlSession;
    }
}
```
key features:
1. Integration with Spring's IoC container
2. Mapper XML file scanning
3. Session management

### Mapper XML Parsing

The framework parse MyBatis mapper XML files:
```java
private Map<String, MapperNode> buildMapperNodes(String filePath) {
    SAXReader saxReader = new SAXReader();
    URL xmlPath = this.getClass().getClassLoader().getResource(filePath);
    try {
        Document document = saxReader.read(xmlPath);
        Element rootElement = document.getRootElement();
        String namespace = rootElement.attributeValue("namespace");

        Iterator<Element> nodes = rootElement.elementIterator();
        while (nodes.hasNext()) {
            Element node = nodes.next();
            String id = node.attributeValue("id");
            String parameterType = node.attributeValue("parameterType");
            String resultType = node.attributeValue("resultType");
            String sql = node.getText();

            MapperNode selectnode = new MapperNode();
            selectnode.setNamespace(namespace);
            selectnode.setId(id);
            selectnode.setParameterType(parameterType);
            selectnode.setResultType(resultType);
            selectnode.setSql(sql);
            selectnode.setParameter("");

            this.mapperNodeMap.put(namespace + "." + id, selectnode);
        }
    } catch (Exception ex) {
        ex.printStackTrace();
    }
    return this.mapperNodeMap;
}
```

This implementation:
1. Uses DOM4J for XML parsing
2. Extracts SQL statements and metadata
3. Creates MapperNode objects

### MapperNode Structure

The **MapperNode** class represents a single SQL statement:
```java
public class MapperNode {
    String namespace;
    String id;
    String parameterType;
    String resultType;
    String sql;
    String parameter;

    // Getters and setters
}
```
Features:
1. Namespace and ID for unique identification
2. Parameter and result type information
3. SQL statement storage
4. Parameter mapping support

### SqlSession Implementation

The **DefaultSqlSession** handles SQL execution:
```java
public class DefaultSqlSession implements SqlSession {
    JdbcTemplate jdbcTemplate;
    SqlSessionFactory sqlSessionFactory;

    @Override
    public Object selectOne(String sqlid, Object[] args, 
            PreparedStatementCallBack pstmtcallback) {
        String sql = this.sqlSessionFactory.getMapperNode(sqlid).getSql();
        return jdbcTemplate.query(sql, args, pstmtcallback);
    }
}
```
Key aspects:
1. SQL statement retrieval
2. Parameter binding
3. Result mapping
4. JDBC template integration

### Integration with Spring Ioc:
The Mybatis integration leverages Spring's IoC container:
```java
@Autowired
JdbcTemplate jdbcTemplate;
```
Benefits:
1. Automatic dependency injection
2. Transaction management
3. Connection pooling
4. Resource management

### XML Configuration
Example mapper XML configuration:
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.UserMapper">
    <select id="findById" parameterType="java.lang.Integer" 
            resultType="com.example.User">
        SELECT * FROM users WHERE id = ?
    </select>
</mapper>
```

### Usage Example
Here's how to use the Mybatis integration:
```java
@Repository
public class UserDao {
    @Autowired
    private SqlSessionFactory sqlSessionFactory;

    public User findById(Integer id) {
        SqlSession session = sqlSessionFactory.openSession();
        try {
            return (User) session.selectOne(
                "com.example.UserMapper.findById",
                new Object[]{id},
                new UserRowMapper()
            );
        } finally {
            session.close();
        }
    }
}
```
### Key Features

1. **Mapper XML Support**
  - XML-Based SQL configuration
  - Dynamic SQL support
  - Parameter mapping
2. **Session Management**
  - Connection handing
  - Transaction boundaries
  - Resource cleanup
3. **Result Mapping**
  - Object mapping
  - Type conversion
  - Collection handing
4. **Spring Integration**
  - IoC container support
  - Transaction management
  - Resource management

### Implementation Details

1. **Mapper Scanning**

```java
private void scanLocation(String location) {
    String sLocation = this.getClass().getClassLoader()
            .getResource("").getPath() + location;
    File dir = new File(sLocation);
    for (File file : dir.listFiles()) {
        if (file.isDirectory()) {
            scanLocation(location + "/" + file.getName());
        } else {
            buildMapperNodes(location + "/" + file.getName());
        }
    }
}
```

2. **SQL Execution**

```java
public Object selectOne(String sqlid, Object[] args, 
        PreparedStatementCallBack pstmtcallback) {
    String sql = this.sqlSessionFactory.getMapperNode(sqlid).getSql();
    return jdbcTemplate.query(sql, args, pstmtcallback);
}
```

### Best Practice
1. **Resource Management**
  - Proper session cleanup
  - Connection pooling
  - Transaction boundaries
2. **Error Handling**
  - SQL exception handling
  - Resource cleanup in finally blocks
  - Proper error propagation
3. **Performance Optimization**
  - Statement caching
  - Connection pooling
  - Batch processing support

### Common Challenges and Solutions
1. **Connection Management**
  - Use connection pooling
  - Implement proper cleanup
  - Handle transaction boundaries
2. **SQL Mapping**
  - Proper parameter binding 
  - Result type conversion
  - Collection handling
3. **Transaction Management**
  - Spring transaction integration
  - Proper isolation levels
  - Rollback handling

### Conclusion
Implementing MyBatis integration in a Spring-like framework provides:
- Clean separation of concerns 
- Flexible SQL mapping 
- Transaction management 
- Resource optimization
Key takeaways:
1. Understanding MyBatis core concepts 
2. Spring integration patterns 
3. Resource management 
4. Performance considerations

This implementation demonstrates how to create a robust ORM framework integration while maintaining simplicity and flexibility.
