---
title: Method-Level Authorization in Spring Boot
tags:
  - Java
  - Spring
  - Security
---

In this post, we'll explore how to use Spring Security to control access both at 
the endpoint and method level using a sample project. 
We'll cover role-based and authority-based security, showing how both can be configured 
and used to enhance your application's overall security posture.

## Spring Security Configuration

To get started, we need to configure Spring Security. Below is the `SecurityConfig` class, which is responsible for setting up the security rules and ensuring proper authentication and authorization for our application. With Spring Security, you can create custom security policies that help protect your application from unauthorized access.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/public/**").permitAll()
                    .requestMatchers("/admin/api/**").hasRole("ADMIN") 
                    .requestMatchers("/api/**").hasRole("USER")
                    .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults());

    return http.build();
}
```

The `SecurityFilterChain` bean is responsible for defining how requests are secured. We start by disabling CSRF protection for simplicity (note that in production, you should consider enabling it for non-API requests). The authorization rules are straightforward: any request to `/public/**` is allowed without authentication, while `/admin/api/**` is restricted to users with the `ADMIN` role. Similarly, `/api/**` endpoints require the `USER` role, and all other requests must be authenticated. This layered approach to authorization allows us to provide clear boundaries for different types of users within our application, ensuring that each user only has access to the features they need.

```java
@Bean
public UserDetailsService userDetailsService() {
    var user1 = User.withUsername("user1")
            .password("{noop}user1")
            .roles("USER")
            .build();

    var admin1 = User.withUsername("admin1")
            .password("{noop}admin1")
            .roles("ADMIN")
            .build();

    var admin2 = User.withUsername("admin2")
            .password("{noop}admin2")
            .authorities("ROLE_ADMIN", "CREATE_ORDER")
            .build();

    return new InMemoryUserDetailsManager(user1, admin1, admin2);
}
```

In this configuration, we set up an `InMemoryUserDetailsManager` with three users: `user1`, `admin1`, and `admin2`. The user `user1` has the role `USER`, while `admin1` has the role `ADMIN`. Additionally, `admin2` has both the `ADMIN` role and an extra authority called `CREATE_ORDER`, which allows them to perform more specific actions, such as creating orders.

In addition to HTTP Basic authentication, this configuration can easily be extended to include OAuth2 or JWT-based authentication to provide more sophisticated security mechanisms. By using role-based access control in combination with more advanced authentication techniques, you can build a highly secure application that meets the requirements of modern software environments.

## Applying Method-Level Authorization

To illustrate how to implement method-level authorization, we have two controllers in our project: `AdminController` and `Controller`. The `AdminController` handles administrative operations related to orders, while the general `Controller` manages regular user orders. By having separate controllers for different roles, we ensure that the application follows the principle of least privilege, where users only access the data they need.

```java
@RestController
@RequestMapping("/admin/api/orders")
public class AdminController {

    @GetMapping
    public String getOrders() {
        return "Admin orders returned";
    }

    @PostMapping
    @PreAuthorize("hasAuthority('CREATE_ORDER')")
    public String createOrder() {
        return "Order created";
    }
}
```

In the `AdminController`, there are two endpoints. The `GET` endpoint returns a list of orders, while the `POST` endpoint is used to create a new order. The `POST` endpoint is protected using the `@PreAuthorize` annotation to ensure that only users with the `CREATE_ORDER` authority can access it.

The `@PreAuthorize` annotation is a powerful feature provided by Spring Security that allows you to specify authorization requirements at the method level. In this case, only users with the `CREATE_ORDER` authority are allowed to create a new order. This allows us to provide more granular control over access, ensuring that sensitive actions are restricted to users with the appropriate permissions.

```java
@RestController
@RequestMapping("/api/orders")
public class Controller {

    @GetMapping
    public String getOrders() {
        return "Orders returned";
    }
}
```

The general `Controller` is used for managing orders accessible to regular users. It has a `GET` endpoint that simply returns a list of orders:

This controller does not have any special method-level security annotations, as it is intended for users with the `USER` role. The security configuration in `SecurityConfig` ensures that only users with the appropriate role can access these endpoints. This approach allows us to simplify the security rules for general users while applying more stringent rules for administrative tasks.

## Response Examples for Endpoint Requests

Here are some examples of the responses for different requests made to the API endpoints:

```bash
GET localhost:8080/api/orders

HTTP 401 Unauthorized
{
  "error": "Unauthorized",
  "message": "Full authentication is required to access this resource"
}
```

```bash
GET localhost:8080/api/orders
Authorization: Basic user1 user1

HTTP 200 OK
Orders returned
```

```bash
GET localhost:8080/admin/api/orders
Authorization: Basic admin1 admin1

HTTP 200 OK
Admin orders returned
```

```bash
POST localhost:8080/admin/api/orders
Authorization: Basic admin1 admin1

HTTP 403 Forbidden
{
  "error": "Forbidden",
  "message": "Access is denied"
}
```

```bash
POST localhost:8080/admin/api/orders
Authorization: Basic admin2 admin2

HTTP 200 OK
Order created
```

## Roles vs Authorities

Roles are a specific type of authority in Spring Security, distinguished by the `"ROLE_"` prefix in their names. For example, `.roles("ADMIN")` is equivalent to `.authorities("ROLE_ADMIN")`.

However, caution is needed when mixing `roles` and `authorities` while configuring user permissions. In the example below, the user will have only the `CREATE_ORDER` authority because `.authorities("CREATE_ORDER")` overrides `.roles("ADMIN")`.

```java
var admin = User.withUsername("admin")
    .password("{noop}admin")
    .roles("ADMIN") // This is translated to "ROLE_ADMIN"
    .authorities("CREATE_ORDER") // This overrides the role
    .build();
```

In such cases, explicitly specify all desired authorities in `.authorities()` to avoid unintentional overrides.

## Additional Considerations for Method-Level Security

Additionally, method-level security annotations like `@PreAuthorize` can be combined with other annotations such as `@PostAuthorize`, `@Secured`, and `@RolesAllowed` to provide even more flexibility. For instance, `@PostAuthorize` can be used to validate the response after the method has executed, which can be helpful in certain scenarios, such as ensuring that a user only sees data they are allowed to access.


## Source Code

You can find the source code for this demo on GitHub: [Spring Security Method-Level Authorization Demo](https://github.com/moham-p/dev-codes/tree/main/spring/security)
