---
title: "Spring Architecture Series-6.Implementing JDBC Module in Spring"
meta_title: ""
description: ""
date: 2025-03-27T00:00:00Z
image: ""
categories: ["Spring", "Java"]
author: "yaruyng"
tags: ["Spring", "Java"]
draft: false
---
Implementing JDBC Module in Spring
<!--more-->

## Introduce
JDBC(Java Database Connectivity) is the standard API for database access in Java.However,working with JDBC can be tedious and error-prone.In this article,i"ll explore how to implement a JDBC module that simplifies database operations, based on my miniSpring project's implementation.

## Core Components
The JDBC module consists of several key components:
```text
src/com/yaruyng/jdbc/
├── core/
│   ├── JdbcTemplate.java
│   ├── RowMapper.java
│   ├── ResultSetExtractor.java
│   ├── StatementCallBack.java
│   ├── PreparedStatementCallBack.java
│   └── ArgumentPreparedStatementSetter.java
├── datasource/
└── pool/
```

## JdbcTemplate:The Core Class
The **JdbcTemplate** class is the central component that simplifies JDBC operations:
```java
public class JdbcTemplate {
  private DataSource dataSource;

  public Object query(StatementCallBack stmtCallBack) {
    Connection con = null;
    Statement stmt = null;
    try {
      con = dataSource.getConnection();
      stmt = con.createStatement();
      return stmtCallBack.doInStatement(stmt);
    } catch (SQLException e) {
      e.printStackTrace();
    } finally {
      try {
        stmt.close();
        con.close();
      } catch (SQLException e) {
      }
    }
    return null;
  }

  public Object query(String sql, Object[] args,
                      PreparedStatementCallBack pstmtCallBack) {
    Connection con = null;
    PreparedStatement pstmt = null;
    try {
      con = dataSource.getConnection();
      pstmt = con.prepareStatement(sql);
      ArgumentPreparedStatementSetter setter =
        new ArgumentPreparedStatementSetter(args);
      setter.setValues(pstmt);
      return pstmtCallBack.doInPreparedStatement(pstmt);
    } catch (SQLException e) {
      throw new RuntimeException(e);
    } finally {
      try {
        pstmt.close();
        con.close();
      } catch (SQLException e) {
      }
    }
  }
}
```
Key features:
1. Resource management
2. Exception handing
3. Connection pooling support
4. Prepared statement support

## Row Mapping
The **RowMapper** interface provides a flexible way to map database rows to objects:
```java
public interface RowMapper<T> {
    T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```
Usage example
```java
public class UserRowMapper implements RowMapper<User> {
    @Override
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        return user;
    }
}
```

## Parameter Handling
The **ArgumentPreparedStatementSetter** class handles parameter binding:
```java
public class ArgumentPreparedStatementSetter {
    private final Object[] args;

    public void setValues(PreparedStatement pstmt) throws SQLException {
        if (this.args != null) {
            for (int i = 0; i < this.args.length; i++) {
                Object arg = this.args[i];
                doSetValue(pstmt, i+1, arg);
            }
        }
    }

    private void doSetValue(PreparedStatement pstmt, 
            int parameterPosition, Object argValue) throws SQLException {
        if (argValue instanceof String) {
            pstmt.setString(parameterPosition, (String)argValue);
        }
        else if (argValue instanceof Integer) {
            pstmt.setInt(parameterPosition, (int)argValue);
        }
        else if (argValue instanceof java.util.Date) {
            pstmt.setDate(parameterPosition, 
                new java.sql.Date(((java.util.Date)argValue).getTime()));
        }
    }
}
```
Features:
1. Type-safe parameter binding
2. Support for common data types
3. Extensible design

## Result Set Extraction
The **ResultSetExtractor** interface provides a way to process result sets:
```java
public interface ResultSetExtractor<T> {
  T extractData(ResultSet rs) throws SQLException;
}
```
Implementation example:
```java
public class RowMapperResultSetExtractor<T> implements ResultSetExtractor<List<T>> {
    private final RowMapper<T> rowMapper;

    public RowMapperResultSetExtractor(RowMapper<T> rowMapper) {
        this.rowMapper = rowMapper;
    }

    @Override
    public List<T> extractData(ResultSet rs) throws SQLException {
        List<T> results = new ArrayList<>();
        int rowNum = 0;
        while (rs.next()) {
            results.add(rowMapper.mapRow(rs, ++rowNum));
        }
        return results;
    }
}
```

## Usage Example
### 1. Simple Query
```java
List<User> users = jdbcTemplate.query(
    "SELECT * FROM users WHERE age > ?",
    new Object[]{18},
    new UserRowMapper()
);
```
### 2. Custom Statement Processing
```java
Object result = jdbcTemplate.query(new StatementCallBack() {
    @Override
    public Object doInStatement(Statement stmt) throws SQLException {
        ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM users");
        if (rs.next()) {
            return rs.getInt(1);
        }
        return 0;
    }
});
```
### 3. Prepared Statement with Callback
```java
Object result = jdbcTemplate.query(
    "UPDATE users SET name = ? WHERE id = ?",
    new Object[]{"John", 1},
    new PreparedStatementCallBack() {
        @Override
        public Object doInPreparedStatement(PreparedStatement pstmt) 
                throws SQLException {
            return pstmt.executeUpdate();
        }
    }
);
```

## Key Features
### 1. Resource Management
  - Automatic connection handling
  - Statement cleanup
  - Exception handling
### 2. Type Safety
  - Generic row mapping
  - Type-safe parameter binding
  - Result type conversion
### 3. Flexibility
  - Custom statement processing
  - Extensible row mapping
  - Configurable result extraction
### 4. Error Handling
  - SQL exception wrapping
  - Resource cleanup
  - Transaction support

## Best Practices
### 1. Connection Management
  - Use connection pooling
  - Proper resource cleanup
  - Transaction boundaries
### 2. Exception Handling
  - Custom exception types
  - Proper error propagation
  - Resource cleanup in finally blocks
### 3. Performance Optimization
  - Statement caching
  - Batch processing
  - Connection pooling

## Common Challenges and Solutions
### 1. Resource Leaks
  - Use try-with-resources
  - Proper cleanup in finally blocks
  - Connection pooling
### 2. Type Conversion
  - Implement type handlers
  - Use prepared statements
  - Handle null values
### 3. Transaction Management
  - Spring transaction integration
  - Proper isolation levels
  - Rollback handling

## Conclusion
Implementing a JDBC module provides:
- Simplified database access
- Type-safe operations
- Resource management
- Error handling
Key takeaways:
1. Understanding JDBC fundamentals 
2. Resource management patterns 
3. Type safety in database operations 
4. Performance optimization techniques

This implementation demonstrates how to create a robust database access layer while maintaining simplicity and flexibility.
