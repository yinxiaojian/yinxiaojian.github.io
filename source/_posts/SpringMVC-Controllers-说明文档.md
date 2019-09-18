---
title: SpringMVC Controllers 说明文档
date: 2018-01-05 09:41:54
tags: Controllers
categories: SpringMVC
---

翻译自官方文档，并加上了自己的理解，解释更加直白。

官方文档地址：[Spring Web MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc)

<!--more-->

### 1 Declaration

使用@Controller注解标记一个类，这个类就是一个SpringMVC Controller对象。

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

在XML文件进行配置，告诉Spring应该到哪里去找Controller控制器，加上如下一行，base-package 即controller所在位置

```java
<context:component-scan base-package="org.example.web"/>
```

完整XML配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```



### 2 Request Mapping

@RequestMapping 注解用于映射Request请求与与controller的处理方法。该注解有多个参数可以配置Request请求的属性，如URL，HTTP method，request parameters，headers，media types。

@RequestMapping 可以用于类也可以用于类方法。当@RequestMapping 标记在Controller 类上的时候，里面使用@RequestMapping 标记的方法的请求地址都是相对于类上的@RequestMapping 而言的；当Controller 类上没有标记@RequestMapping 注解时，方法上的@RequestMapping 都是绝对路径。最终路径都是相对于跟路径"/"的。通过这种组合的方法可以限制Request的匹配。

在SpringMVC 4.3 引入了组合注解来简化@RequestMapping的写法。

* @GetMapping 
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping

如@GetMapping等价于@RequestMapping(method = RequestMethod.GET)，组合注解通常用于method上，如下

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

#### 2.1 URI patterns

我们可以使用通配符去匹配request

* `?`匹配一个字符
* `*`匹配一个路径段中的零个或多个字符
* `**`匹配零个或多个路径段

可以使用@PathVariable来声明URI变量并获取其值:

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

URI变量即可以在类层级声明，也可以在方法层级声明:

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable("ownerId") Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI变量会进行自动类型转换或者抛出`TypeMismatchException`异常，对于简单的类型，如`int, long, Data`，默认自动转换，对于复杂类型在此不做详解。

URI变量作为参数时，有两种声明方式：

1. 显性声明，如上代码块： `@PathVariable("ownerId")`，这种声明方式明确规定使用的是URI模版里的ownerId变量。
2. 直接使用@PathVariable，如上代码块`@PathVariable Long petId`，这种情况下会默认去URI模版寻找跟参数名相同的变量，但只能在使用debug模式才可以。

Request还支持正则匹配，语法格式：`{varName:regex}`，用regex声明了一个URI变量varName，例如：

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

#### 2.2 Pattern comparison

当有多个patterns（模版）匹配到URL时，通过`AntPathMatcher.getPatternComparator(String path)`去获取最合适的patterns。

对于每一个pattern，根据URI变量和通配符的个数计算出分数，分数越低优先度越高。相同分数则较长者优先度高。

默认映射模版`/**`不参与比较，优先度最低。

#### 2.3 Matrix variables

Matrix variables可以出现在任意路径段，每个matrix variable由 ";" 分割，例如`/cars;color=red;year=2012`。多个值既可以用 "," 分割，如`color=red,green,blue`，也可以重复变量名，如`color=red;color=green;color=blue`。

如果一个URL可能含有matrix variables，那么请求映射模版必须使用URI模版去表示。这样可以确保匹配正确，即使matrix variables的位置不固定或不存在。

如下例子，获取matrix variable "q"

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

多个路径段包含matrix variables的情况：

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

可以设置matrix variable的required属性，`required = false`表示该参数不是必须存在的，同时可以设置defaultValue赋予默认值。如下：

```Java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

可以将所有的matrix variables放置于一个Map中：

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId"") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

最后注意：matrix variables默认是不启用的，因此我们需要在xml文件中进行配置:

```Xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven enable-matrix-variables="true"/>

</beans>
```

#### 2.4 Consumable media types

利用`Content-Type`对请求匹配范围进行限制，从而缩小请求的映射范围。

```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

consumes 属性支持否定表达，`!text/plain`表示除了 "text/plain" 的所有content type。

consumes可以声明在class层级，与其他request mapping attributes不同的是，当声明在class层级时，method层级的consumes属性会覆盖而不是扩展class层级的声明。

#### 2.5 Producible media types

利用`Accept`对请求匹配范围进行限制，从而缩小请求的映射范围。类似2.4:

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json;charset=UTF-8")
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

#### 2.6 params, method, headers

* params 属性用于指定请求参数
* method 属性用于限制能够访问的方法类型 //和组合注解@GetMapping等类似
* headers 属性用于指定请求头信息

三者都可以缩小请求的映射范围，支持否定表达。

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue")
public void findPet(@PathVariable String petId) {
    // ...
}
@GetMapping(path = "/pets", headers = "myHeader=myValue")
public void findPet1(@PathVariable String petId) {
    // ...
}
//和findPet1等价
@RequestMapping(path = "/pets", headers = "myHeader=myValue", method = RequestMethod. GET)
public void findPet2(@PathVariable String petId) {
    // ...
}
```



### 3 Handler Methods

#### 3.1 Method Arguments

> 引用自官方文档

| Controller method argument               | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| `WebRequest`, `NativeWebRequest`         | Generic access to request parameters, request & session attributes, without direct use of the Servlet API. |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | Choose any specific request or response type — e.g. `ServletRequest`, `HttpServletRequest`, or Spring’s `MultipartRequest`, `MultipartHttpServletRequest`. |
| `javax.servlet.http.HttpSession`         | Enforces the presence of a session. As a consequence, such an argument is never `null`.**Note:** Session access is not thread-safe. Consider setting the`RequestMappingHandlerAdapter`'s "synchronizeOnSession" flag to "true" if multiple requests are allowed to access a session concurrently. |
| `javax.servlet.http.PushBuilder`         | Servlet 4.0 push builder API for programmatic HTTP/2 resource pushes. Note that per Servlet spec, the injected `PushBuilder` instance can be null if the client does not support that HTTP/2 feature. |
| `java.security.Principal`                | Currently authenticated user; possibly a specific `Principal` implementation class if known. |
| `HttpMethod`                             | The HTTP method of the request.          |
| `java.util.Locale`                       | The current request locale, determined by the most specific `LocaleResolver` available, in effect, the configured `LocaleResolver`/`LocaleContextResolver`. |
| Java 6+: `java.util.TimeZone`Java 8+: `java.time.ZoneId` | The time zone associated with the current request, as determined by a `LocaleContextResolver`. |
| `java.io.InputStream`, `java.io.Reader`  | For access to the raw request body as exposed by the Servlet API. |
| `java.io.OutputStream`, `java.io.Writer` | For access to the raw response body as exposed by the Servlet API. |
| `@PathVariable`                          | For access to URI template variables. See [URI patterns](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-uri-templates). |
| `@MatrixVariable`                        | For access to name-value pairs in URI path segments. See [Matrix variables](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-matrix-variables). |
| `@RequestParam`                          | For access to Servlet request parameters. Parameter values are converted to the declared method argument type. See [@RequestParam](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestparam). |
| `@RequestHeader`                         | For access to request headers. Header values are converted to the declared method argument type. See [@RequestHeader](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestheader). |
| `@RequestBody`                           | For access to the HTTP request body. Body content is converted to the declared method argument type using `HttpMessageConverter`s. See [@RequestBody](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestbody). |
| `HttpEntity<B>`                          | For access to request headers and body. The body is converted with `HttpMessageConverter`s. See [HttpEntity](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-httpentity). |
| `@RequestPart`                           | For access to a part in a "multipart/form-data" request. See [@RequestPart](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-multipart-forms-non-browsers) and [Multipart requests](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-multipart). |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | For access and updates of the implicit model that is exposed to the web view. |
| `RedirectAttributes`                     | Specify attributes to use in case of a redirect — i.e. to be appended to the query string, and/or flash attributes to be stored temporarily until the request after redirect. See [Redirect attributes](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-redirecting-passing-data) and [Flash attributes](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-flash-attributes). |
| Command or form object (with optional `@ModelAttribute`) | Command object whose properties to bind to request parameters — via setters or directly to fields, with customizable type conversion, depending on `@InitBinder` methods and/or the HandlerAdapter configuration (see the `webBindingInitializer` property on`RequestMappingHandlerAdapter`).Command objects along with their validation results are exposed as model attributes, by default using the command class name - e.g. model attribute "orderAddress" for a command object of type "some.package.OrderAddress". `@ModelAttribute` can be used to customize the model attribute name. |
| `Errors`, `BindingResult`                | Validation results for the command/form object data binding; this argument must be declared immediately after the command/form object in the controller method signature. |
| `SessionStatus`                          | For marking form processing complete which triggers cleanup of session attributes declared through a class-level `@SessionAttributes`annotation. |
| `UriComponentsBuilder`                   | For preparing a URL relative to the current request’s host, port, scheme, context path, and the literal part of the servlet mapping also taking into account `Forwarded` and `X-Forwarded-*` headers. |
| `@SessionAttribute`                      | For access to any session attribute; in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes`declaration. |
| `@RequestAttribute`                      | For access to request attributes.        |

#### 3.2 Return Values

> 引用自官方文档

| Controller method return value           | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| `@ResponseBody`                          | The return value is converted through `HttpMessageConverter`s and written to the response. See [@ResponseBody](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-responsebody). |
| `HttpEntity<B>`, `ResponseEntity<B>`     | The return value specifies the full response including HTTP headers and body be converted through `HttpMessageConverter`s and written to the response. See [HttpEntity](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-httpentity). |
| `HttpHeaders`                            | For returning a response with headers and no body. |
| `String`                                 | A view name to be resolved with `ViewResolver`'s and used together with the implicit model — determined through command objects and `@ModelAttribute`methods. The handler method may also programmatically enrich the model by declaring a `Model`argument (see above). |
| `View`                                   | A `View` instance to use for rendering together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (see above). |
| `java.util.Map`, `org.springframework.ui.Model` | Attributes to be added to the implicit model with the view name implicitly determined through a `RequestToViewNameTranslator`. |
| `ModelAndView` object                    | The view and model attributes to use, and optionally a response status. |
| `void`                                   | A method with a `void` return type (or `null` return value) is considered to have fully handled the response if it also has a `ServletResponse`, or an `OutputStream` argument, or an `@ResponseStatus` annotation. The same is true also if the controller has made a positive ETag or lastModified timestamp check (see [@Controller caching](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-caching-etag-lastmodified) for details).If none of the above is true, a `void`return type may also indicate "no response body" for REST controllers, or default view name selection for HTML controllers. |
| `Callable<V>`                            | Produce any of the above return values asynchronously in a Spring MVC managed thread. |
| `DeferredResult<V>`                      | Produce any of the above return values asynchronously from any thread — e.g. possibly as a result of some event or callback. |
| ListenableFuture<V>, java.util.concurrent.CompletionStage<V>, java.util.concurrent.CompletableFuture<V> | Alternative to `DeferredResult` as a convenience for example when an underlying service returns one of those. |
| `ResponseBodyEmitter`, `SseEmitter`      | Emit a stream of objects asynchronously to be written to the response with`HttpMessageConverter`'s; also supported as the body of a `ResponseEntity`. |
| `StreamingResponseBody`                  | Write to the response `OutputStream` asynchronously; also supported as the body of a`ResponseEntity`. |
| Reactive types — Reactor, RxJava, or others via `ReactiveAdapterRegistry` | Alternative to ``DeferredResult`with multi-value streams (e.g. `Flux`, `Observable`) collected to a `List`.For streaming scenarios — .e.g. `text/event-stream`, `application/json+stream`,`SseEmitter` and `ResponseBodyEmitter` are used instead, where `ServletOutputStream` blocking I/O is performed on a Spring MVC managed thread and back pressure applied against the completion of each write.See [Reactive return values](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types). |
| Any other return type                    | A single model attribute to be added to the implicit model with the view name implicitly determined through a `RequestToViewNameTranslator`; the attribute name may be specified through a method-level `@ModelAttribute` or otherwise a name is selected based on the class name of the return type. |

#### 3.3 @RequestParam

使用 @RequestParam 绑定 HttpServletRequest 请求参数到控制器方法参数：

```java
@Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

绑定的参数默认必须存在，可以通过required属性修改，如`@RequestParam(name="id", required=false)`。

如果控制器方法参数类型不是string，将会执行自动类型转换。

#### 3.4 @RequestHeader

使用@RequestHeader绑定绑定 HttpServletRequest 头信息到控制器方法参数。

一个request header例子:

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

下面的例子演示了如何获取 `Accept-Encoding`和 `Keep-Alive` 的值：

```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}
```

如果控制器方法参数类型不是string，将会执行自动类型转换。

#### 3.5 @CookieValue

使用@CookieValue绑定绑定 HttpServletRequest 的cookie信息到控制器方法参数。

假设http request中有如下cookie信息：

```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

下面的例子演示了如何获取JESSIONID的值：

```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie) {
    //...
}
```

#### 3.6 @ModelAttribute

//TODO