GateWay是Zuul的升级版和替代品，比Zuul2更早使用Netty实现异步IO，从而实现了一个简单、比Zuul1.x更高效的，与Spring Cloud紧密配合的API网关。

SpringCloudGateWay里明确的区分了Router和Filter，并且一个很大的特点是内置了非常多的开箱即用功能，并且都可以通过SpringBoot配置或者手动编码链式调用来使用。比如内置了10种Router，使得我们可以直接配置一下就可以随心所欲的根据Header、或者Path、或者Host、或者Query来做路由。比如区分了一版Filter和全局Filter，内置了20种Filter和9中全局Filter，也都可以直接用。当然自定义Filter也非常方便。

#### 重要概念

- Route（路由）：这是网关的基本构建块。他由一个ID，一个目标URL，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
- Predicate（断言）：输入类型是一个ServerWebExchange。我们可以使用它来匹配来自HTTP请求的任何内容，例如headers或者参数。
- Filter（过滤器）：Gateway中的Filter分为两种类型的Filter，分别是Gateway Filter和Global Filter。过滤器Filter将会对请求和响应进行修改处理。



#### 功能特性

- 基于Spring Framework5，Project Reactor和Spring Boot 2.0进行构建
- 动态路由：能够匹配任何请求属性
- 集成Spring Cloud服务发现功能
- 可以对路由指定Predicate（断言）和Filter（过滤器）
- 易于编写的Predicate（断言）和Filter（过滤器）
- 集成Hystrix的断路器功能
- 请求限流功能
- 支持路径重写



#### Predicate

```yaml
spring:  
    cloud:    
        gateway:      
            routes:      
            - id: after_route        
              uri: http://www.google.com        
              predicates:  
              #After Route Predicate Factory:时间作为匹配规则，只要当前时间大于设定时间，路由才会匹配请求
              - After=2018-12-25T14:33:47.789+08:00
			  #Before Route Predicate Factory:时间作为匹配规则，只要当前时间小于设定时间，路由才会匹配请求
              - Before=2018-12-25T14:33:47.789+08:00
              #Between Route Predicate Factory：两个时间作为匹配规则，只要当前时间大于第一个设定时间，并小于第二个设定时间，路由才会匹配请求
              - Between=2018-12-25T14:33:47.789+08:00, 2018-12-26T14:33:47.789+08:00
			  #Cookie Route Predicate Factory使用的是cookie名字和正则表达式的value作为两个输入参数，请求的cookie需要匹配cookie名和符合其中value的正则，匹配请求存在cookie名为cookiename，cookie内容匹配cookievalue的，转发到google
			  - Cookie=cookiename, cookievalue
			  #Header Route Predicate Factory路由匹配存在名为X-Request-Id，内容为数字的header的请求
			  - Header=X-Request-Id, \d+
			  #Host Route Predicate Factory匹配Host
			  - Host=**.somehost.org,**.anotherhost.org
			  #Method Route Predicate Factory通过HTTP的method来匹配路由。
			  - Method=GET
			  # Path Route Predicate Factory使用Path列表作为参数，使用Spring的PathMatcher匹配path，可以设置可选变量，可以匹配/foo/1或/foor/bar或者/foo/baz等其中segment变量可以通过下面方式获取
			  - Path=/foo/{segment},/bar/{segment}
			  #Query Route Predicate Factory可以通过一个或两个参数来匹配路由，一个是查询的name，一个是查询的正则value
			  - Query=baz
			  #RemoteAddr Route Predicate Factory通过五类别域间路由列表匹配路由
			  - RemoteAddr=192.168.1.1/24
```



#### GateWay Filter

```yaml
spring:  
	cloud:    
		gateway:   
        	routes: 
            - id: add_request_header_route  
              uri: http://www.google.com  
              filters:
              #AddRequestHeader GatewayFilter Factory通过配置name和value可以增加请求的header,匹配的请求，会额外添加X-Request-Foo:Bar的header
              - AddRequestHeader=X-Request-Foo, Bar
              #AddRequestParameter GatewayFilter Factory通过配置name和value可以增加请求参数，对匹配的请求，会额外增加foo=bar的请求参数
              - AddRequestParameter=foo,bar
              #AddResponseHeader GatewayFilter Factory通过配置name和value可以增加响应的header，对匹配的请求，响应返回时会额外增加X-Response-Foo:Bar的header返回。
              #Hystrix GatewayFilter Factory，项目里面引入spring-cloud-starter-netflix-hystrix依赖，并且提供HystrixCommand的名字，即可生效Hystrix GatewayFilter。剩下的过滤器，则会包装在名为myCommandName的HystrixCommand中运行
              - Hystrix=myCommandName
```

