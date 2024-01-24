---
toc: True
comments: True
layout: notebook
hide: True
title: Thymeleaf Lesson With Syntax
type: hacks
---

# What is ThymeLeaf 
Thymeleaf is a template engine for server-side Java applications to create to create pages that can be displayed with within the browser. It can 
process the following templates: 

- HTML 
- XML 
- JavaScript 
- CSS
- Plain text 

It is most commonly used to generate HTML content for websites. Thymeleaf templates are located within the `src/main/resources/templates` directory.

## Structure of Thymeleaf
The templates used by Thymeleaf are just HTML5 files with extra properties added for dynamic content. 

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>This is the title</title>
</head>
<body>
    <h1 th:text="${message}">hello</h1>
</body>
</html>
```

the code defines a simple webpage with a title and a heading. The heading's text is set dynamically using Thymeleaf's syntax.

## Syntax of Thymeleaf
- The basic syntax of Thymeleaf are from the 5 types of basic expressions 
- `${}` - Variable Expressions
- `*{}` - Selection Expressions
- `#{}` - Message Expressions
- `@{}` - Link (URL) Expressions
- `~{}` - Fragment Expressions

### Variable Expressions
- Variable expressions are when integrated with Spring are Spring Expression Language that operate on context variables. 

A context variable could look like `${session.user.name}`


### Selection Expressions

- Selection expressions are similar to variable expressions but execute on the previous object instead of the entire context. 

A selection expression could look like `*{title}`

where it would act on a object specified by `th.object`.  

```Html
<div th:object="${book}">
  ...
  <span th:text="*{title}">...</span>
  ...
</div>
```

### Message Expressions
- Message expressions are used to be abel to access messages from external sources. These are most commonly from .properties files where they can be referenced through using an optional key and applying a set of parameters. 

A message expression could look like `#{title}` You can also use variable expressions within the message expression such as `#{message.entrycreated(${entryId})}`

where it would act on a object specified by `th.object`.  As seen below. 

```HTML
<table>
  ...
  <th th:text="#{header.address.city}">...</th>
  <th th:text="#{header.address.country}">...</th>
  ...
</table>
```

### Link Expressions
- Link expressions are used to create URLs add context to them which is known as URL rewriting. 
- So for instance if you have a shopping endpoint you may use an expression 

```html
<a th:href="@{/order/list}">...</a>
```
- While also being able to have parameters within the URL such as 

```html
<a th:href="@{/order/details(id=${orderId},type=${orderType})}">...</a>
```

- Link expressions also allow for being relative to not add application context through for example 

```html
<a th:href="@{../order/list}">...</a>
```

- Or Server relative with no app context 

```html
<a th:href="@{~/order/list}">...</a>
```
- Or protocol relative using http or https to display the page within the browser. 

```html
<a th:href="@{/grocery.checkout.com/cart}">...</a>
```

- Or absolute link expressions like 

```html
<a th:href="@{http://www.mycompany.com/main}">...</a>
```

### Fragment Expressions


### Attributes

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

# Implementing CRUD with Thymeleaf and Spring 

Thymeleaf allows for seamless use of Spring to easily create CRUD applications through a layer of standard JPA-based CRUD repositories.

## Integrating Thymeleaf with Spring with Maven
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
</dependencies>

```
- Add the following dependencies to your pom.xml file
The other thymeleaf dependencies you need should already be there. 

## Add the Domain Layer
For simplicity’s sake, this layer will include one single class that will be responsible for modeling User entities:


```python
@Entity
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    
    @NotBlank(message = "Name is mandatory")
    private String name;
    
    @NotBlank(message = "Email is mandatory")
    private String email;

    // standard constructors / setters / getters / toString
}
```

## The Repository Layer

Spring Data JPA is a key component of Spring Boot’s spring-boot-starter-data-jpa that makes it easy to add CRUD functionality through a powerful layer of abstraction placed on top of a JPA implementation. This abstraction layer allows us to access the persistence layer without having to provide our own DAO implementations from scratch.

To provide our application with basic CRUD functionality on User objects, we just need to extend the CrudRepository interface:


```python
@Repository
public interface UserRepository extends CrudRepository<User, Long> {}
```

## The Controller Layer

Thanks to the layer of abstraction that spring-boot-starter-data-jpa places on top of the underlying JPA implementation, we can easily add some CRUD functionality to our web application through a basic web tier.

In our case, a single controller class will suffice for handling GET and POST HTTP requests and then mapping them to calls to our UserRepository implementation.

Let’s start with the controller’s showSignUpForm() and addUser() methods.

The former will display the user signup form, while the latter will persist a new entity in the database after validating the constrained fields.

If the entity doesn’t pass the validation, the signup form will be redisplayed.

Otherwise, once the entity has been saved, the list of persisted entities will be updated in the corresponding view:


```python
@Controller
public class UserController {
    
    @GetMapping("/signup")
    public String showSignUpForm(User user) {
        return "add-user";
    }
    
    @PostMapping("/adduser")
    public String addUser(@Valid User user, BindingResult result, Model model) {
        if (result.hasErrors()) {
            return "add-user";
        }
        
        userRepository.save(user);
        return "redirect:/index";
    }

    // additional CRUD methods
}
```

Mapping for the /index URL:


```python
@GetMapping("/index")
public String showUserList(Model model) {
    model.addAttribute("users", userRepository.findAll());
    return "index";
}
```

Within the UserController, we will also have the showUpdateForm() method, which is responsible for fetching the User entity that matches the supplied id from the database.

If the entity exists, it will be passed on as a model attribute to the update form view.

So, the form can be populated with the values of the name and email fields:


```python
@GetMapping("/edit/{id}")
public String showUpdateForm(@PathVariable("id") long id, Model model) {
    User user = userRepository.findById(id)
      .orElseThrow(() -> new IllegalArgumentException("Invalid user Id:" + id));
    
    model.addAttribute("user", user);
    return "update-user";
}
```

Finally, we have the updateUser() and deleteUser() methods within the UserController class.

The first one will persist the updated entity in the database, while the last one will remove the given entity.

In either case, the list of persisted entities will be updated accordingly:


```python
@PostMapping("/update/{id}")
public String updateUser(@PathVariable("id") long id, @Valid User user, 
  BindingResult result, Model model) {
    if (result.hasErrors()) {
        user.setId(id);
        return "update-user";
    }
        
    userRepository.save(user);
    return "redirect:/index";
}
    
@GetMapping("/delete/{id}")
public String deleteUser(@PathVariable("id") long id, Model model) {
    User user = userRepository.findById(id)
      .orElseThrow(() -> new IllegalArgumentException("Invalid user Id:" + id));
    userRepository.delete(user);
    return "redirect:/index";
}
```

## Adding the View Layer

Under the src/main/resources/templates folder, we need to create the HTML templates required for displaying the signup form and the update form as well as rendering the list of persisted User entities.

As stated in the introduction, we’ll use Thymeleaf as the underlying template engine for parsing the template files.

Here’s the relevant section of the add-user.html file:
```
<form action="#" th:action="@{/adduser}" th:object="${user}" method="post">
    <label for="name">Name</label>
    <input type="text" th:field="*{name}" id="name" placeholder="Name">
    <span th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
    <label for="email">Email</label>
    <input type="text" th:field="*{email}" id="email" placeholder="Email">
    <span th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></span>
    <input type="submit" value="Add User">   
</form>
```

And here's how the update-user.html file looks like:

```
<form action="#" 
  th:action="@{/update/{id}(id=${user.id})}" 
  th:object="${user}" 
  method="post">
    <label for="name">Name</label>
    <input type="text" th:field="*{name}" id="name" placeholder="Name">
    <span th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
    <label for="email">Email</label>
    <input type="text" th:field="*{email}" id="email" placeholder="Email">
    <span th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></span>
    <input type="submit" value="Update User">   
</form>
```

Finally, we have the index.html file that displays the list of persisted entities along with the links for editing and removing existing ones:

```
<div th:switch="${users}">
    <h2 th:case="null">No users yet!</h2>
        <div th:case="*">
            <h2>Users</h2>
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Edit</th>
                        <th>Delete</th>
                    </tr>
                </thead>
                <tbody>
                <tr th:each="user : ${users}">
                    <td th:text="${user.name}"></td>
                    <td th:text="${user.email}"></td>
                    <td><a th:href="@{/edit/{id}(id=${user.id})}">Edit</a></td>
                    <td><a th:href="@{/delete/{id}(id=${user.id})}">Delete</a></td>
                </tr>
            </tbody>
        </table>
    </div>      
    <p><a href="/signup">Add a new user</a></p>
</div>

```
