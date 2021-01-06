SpringMVC接收HTTP请求参数，需要HttpRequest对HTTP请求进行参数解析，再结合框架提供的注解从而确定参数的绑定规则



## HTTP请求

HTTP请求中对参数的传递有多个影响因素，通常需要配合使用



### 1. 参数位置

**HTTP URL** - 通常以QueryString、PathVariable形式。前者是在URL中问号“?”后面的参数，K=V结构，参数之间使用“&”连接；后者则是URI的一部分，类似文件树的规则来定位资源

**HTTP Header** - 用户在header中的自定义项，K: V结构，注意要避免使用协议中的常用字段

**HTTP Body** - 主要用于传输数据的部分，可传输多种数据类型，通常根据Header中Content-Type来判断



### 2. HTTP方法

**GET** - 主要用于获取资源，参数传递主要通过URL、Header

**POST/DELETE/PUT** - 对资源进行事务操作，参数传递可通过URL、Header、Body



### 3. Content-Type

> HTTP协议的保留header项，主要用以声明body中的数据类型

**application/x-www-form-urlencoded** - HTTP请求的默认类型。普通form表单，可以理解为是在HTTP Body中的QueryString，K-V结构，参数之间使用“&”连接。在Java servlet中，与QueryString一起组成了`HttpRequest.parameters`部分，解析机制由容器实现。Tomcat中规定该表单参数必须以HTTP POST方法请求，否则忽略

**multipart/form-data** - 复合表单，可以传递多种数据类型，使用`boundary`将表单中各个字段分割成独立的part，每个part使用`Content-Disposition`字段记录该part信息，其中`name`字段为该part的key，对于文件类型还可以有`filename`字段记录信息。

form-data算是以boundary为分隔符的`List<Part>`结构，每个part算是一个K=V结构。可以上传多文件，或文件键值混合等等

> 此时Content-Type值为：multipart/form-data;boundary={随机字符串}
>
> 在body中，以--{boundary}\r\n为分隔符

**application/json、application/xml** - 结构化数据，通常采用POST请求并通过HTTP Body传输数据

**application/octet-stream** - 流数据，通常用于传输单文件

**others** - 其他类型（例如多媒体、纯文本等）



## SpringMVC注解

参数类型包含：基本类型、对象类型、文件类型、内嵌对象



### 1. 无注解

> 解析HTTP Request通过getParameter()可以获取到的K-V结构参数

**基本类型：**根据接口参数名称与HTTP参数进行匹配，参数可以有多个

```java
@RequestMapping(value = "/param")
public void param(String name，int age) {
  	System.out.println("name: " + name + ", age: " + age);
}
```

---

**对象类型：**根据对象内属性名称与HTTP参数进行匹配，参数可以有多个（可以将对象理解为基本类型参数的集合，在解析时将对象内所有属性平铺展开）

```java
@RequestMapping("/object")
public void object(StudentDTO student, UserDTO user) {
		System.out.println(student.toString() + "\r\n" + user.toString());
}
```



### 2. @RequestParam

**基本类型：**与无注解行为一致，但可以自定义参数名称以及是否为必传

```java
@RequestMapping(value = "/param")
public void param(@RequestParam(name = "my_name", required = false) String name，int age) {
  	System.out.println("name: " + name + ", age: " + age);
}
```

---

**对象类型：**不适用



### 3. @RequestHeader

> 解析HTTP Header中的参数

使用规则同`@RequestParam`



### 4. @CookieValue

> 解析HTTP Header中Cookie参数

使用规则同`@RequestParam`



### 5. @PathVariable

> 解析URI中通过占位符指定的参数，例如：/user/{id}

使用规则同`@RequestParam`



### 6. @RequestBody

> 解析HTTP Body中的内容，接口参数中只能有一个Body注解

**基本类型：**基本不适用，可以设置`Content-Type`为`text/plain`或`x-www-form-urlencoded`，接口参数为String类型

---

**对象类型：**根据Content-Type值选择对应解析器

```java
@RequestMapping(value = "/body")
public void body(@RequestBody UserDTO user) {
    System.out.println(user.toString());
}
```



### 7. @RequestPart

> 按照multipart/form-data形式解析HTTP Body中的内容

**基本类型：**同`@RequestParam`

---

**对象类型：**基本不适用

---

**文件类型：**接口参数类型为`MultipartFile`

```java
@RequestMapping(value = "/file")
public void file(@RequestPart(value = "text") MultipartFile file) throws IOException {
    StringBuilder stringBuilder = new StringBuilder();
    InputStreamReader streamReader = new InputStreamReader(file.getInputStream());
    BufferedReader reader = new BufferedReader(streamReader);
    String line = null;
    while ((line = reader.readLine()) != null) {
      	stringBuilder.append(line);
    }
    System.out.println(stringBuilder.toString());
}
```



### 8. @InitBinder

在当前Controller范围内添加绑定器并设置绑定规则，被注解标识的方法只有一个参数且类型为`org.springframework.web.bind.WebDataBinder`

可以注册自定义编辑器、设置参数前缀、添加参数校验器等等



### 9. @ModelAttribute

> @ModelAttribute机制等价于@RequestMapping，但先于后者执行，且不能绑定到指定接口上

该注解主要用于在跳转到动态页面前传入数据，被标注的方法可以使用`Model`类型的参数进行数据添加，并且会传递到动态页面中，也可以无参有返回值，该返回值的key为注解指定name

- 标注在方法上：在当前Controller范围内，设置接口执行之前的逻辑

- 标注在参数上：将对应的数据绑定到指定的参数上



## 其他

### 1. Form-data其他解析方式

`@RequestParam`也可以解析`multipart/form-data`，但需要额外配置`HttpMessageConverter`，有两种方式：

* `StandardServletMultipartResolver` + `Servlet 3.0+`
* `CommonsMultipartResolver` + `commons-fileupload`



### 2. 强烈建议指定参数名

当注解中未指定name时，SpringMVC会根据参数名进行匹配，虽然便捷了开发，但在第一次调用时，SpringMVC必须利用`asm、javassist`等组件通过分析字节码中本地变量表获取参数名，并缓存到内存中，既降低效率又浪费资源



### 3. InitBinder + ModelAttribute

利用InitBinder配置默认前缀，ModelAttribute指定使用绑定器可以实现接口中多个对象接收请求参数

```java
@InitBinder("u")
public void u(WebDataBinder binder) {
  	binder.setFieldDefaultPrefix("u.");
}

@InitBinder("s")
public void s(WebDataBinder binder) {
  	binder.setFieldDefaultPrefix("s.");
}

@RequestMapping("/multi")
public void multi(@ModelAttribute("s") StudentDTO student, @ModelAttribute("u") UserDTO user) {
  	System.out.println(student.toString() + "\r\n" + user.toString());
}
```


