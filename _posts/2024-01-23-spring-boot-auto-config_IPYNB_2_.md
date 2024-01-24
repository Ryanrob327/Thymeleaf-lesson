---
toc: True
comments: True
hide: True
layout: notebook
title: spring boot auto config
type: hacks
---

# Auto-Configuration:

In Spring Boot, Auto-Configuration automatically configures beans based on the dependencies present in the classpath. Thymeleaf Auto-Configuration simplifies the configuration of Thymeleaf-related beans, making it easy to integrate Thymeleaf into your Spring Boot application without extensive manual setup.

### Steps to Enable Thymeleaf Auto-Configuration:

1. Ensure that you have the Thymeleaf dependency in your pom.xml:
   
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2. HTML Templates in src/main/resources/templates:

Thymeleaf looks for templates in the src/main/resources/templates directory by default. Create HTML templates in this directory.

3. Application Properties:

Spring Boot provides sensible default configurations for Thymeleaf. You can customize properties in application.properties if needed. For example:

```
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

This configuration sets the template prefix and suffix.


```python

```
