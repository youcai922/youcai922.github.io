#### springmvc转发和重定向

- 转发（forward）：客户端发出请求，服务器接收到该请求，并直接做出相应的处理后进行页面跳转（在服务器端直接完成）。
- 重定向（redirect）：客户端先发出请求，服务器接收到该请求，接收到之后并不直接做处理，而是在返回给浏览器，在让游览器发出一次请求到服务器端，并找到需要处理的页面，找到之后在返回给客户端（浏览器来完成）。

原生实现方式：

```java
//原生转发：
return request.getRequestDispatcher("路径").forward(request, response);
//原生重定向：
return response.sendRedirect("路径");
```

SpringMVC

```java
//SpringMvc转发：
return "forward:/路径";
//SpringMvc重定向：
return "redirect:/路径";
```





