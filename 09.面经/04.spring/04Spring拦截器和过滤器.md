#### SpringBoot的过滤器

```java
//方式一，通过WebFilter注解实现，并且需要在启动类上注明过滤器的包
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

//过滤器
@WebFilter(urlPatterns = "/myfilter")
public class MyFilter implements Filter {
    //容器启动的时候，过滤器初始化的时候调用，只会调用一次，该方法执行失败，则过滤器失效
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    //每一次都会调用该方法
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("---------您已进入过滤器----------");
        filterChain.doFilter(servletRequest,servletResponse);
    }
    //容器销毁的时候执行，也只会执行一次
    @Override
    public void destroy() {
    }
}

@SpringBootApplication
@ServletComponentScan(basePackages = "com.test.springboot.filter")  //第一种方式
public class SpringbootTest14Application {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootTest14Application.class, args);
    }
}
```

```java
//方法二，通过实现Filter接口，并且配置
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;
//过滤器
public class MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("---------您已进入过滤器222----------");
        filterChain.doFilter(servletRequest,servletResponse);
    }
}
//配置类
package com.test.springboot.config;
 
import com.test.springboot.filter2.MyFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
@Configuration  //定义此类为配置类
public class FilterConfig {
    @Bean
    public FilterRegistrationBean myFilterRegistrationBean(){
        FilterRegistrationBean filterRegistrationBean=new FilterRegistrationBean(new MyFilter());
        //添加过滤路径，所有的/user/*的请求都进行处理
        filterRegistrationBean.addUrlPatterns("/user/*");
        return filterRegistrationBean;
    }
}
//Controller
package com.test.springboot.controller;
 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
 
@Controller
public class MyController {
    @RequestMapping("/user/detail")
    public @ResponseBody String userDetail(){
        return "/user/detail";
    }
}
```

#### SpringBoot拦截器

一个请求可以被多个拦截器拦截，执行顺序根据拦截器的声明顺序依次执行

```java
//实现HandlerIntercepter接口
@Component
class InterceptorTest implements HandlerInterceptor{
    //preHandle方法在处理请求前执行，返回true表示放行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }
    //处理请求之后执行
	@Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }
    //响应后执行（主要用于资源清理）
	@Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}
//注册拦截器
@Component
public class InterceptorConfig implements WebMvcConfigurer{//MVC配置类接口
    @Autowired
    Interceptor interceptor;
    public void addInterceptor(InterceptorRegistry interceptorRegistry){//InterceptorRegistry拦截器注册对象
        interceptorRegistry.addInterceptor(interceptor)//注册自定义拦截器
            .addPathPatterns("/test/login")//拦截的路径
            .excludePathPatterns("/test/visit");//不拦截的路径
    }
}
```



#### SpringBoot的拦截器和过滤器对比

- 实现原理不同：过滤器是基于函数回调，拦截器是基于java反射机制。
- 使用范围不同：过滤器依赖于Servlet容器，拦截器不依赖于servlet容器。因为Filter是Servlet规范中规定的，所有只能用于WEB中；而拦截器是Spring的组件，既可以用于WEB，也可以用于Application、Swing中。
- 触发时机不同：Tomcat->Filter->Servlet->Interceptor->Controller
  - 过滤器是在请求进入tomcat容器之后，但在请求servlet容器之前进行预处理的。请求返回结果也是，是在servlet处理完后，再返回给前端的。
  - 拦截器可以深入到方法的前后，抛出异常前后等更深层次的程度进行处理，所以在Spring框加中优先使用拦截器。
- 拦截的请求范围不同：过滤器可以对所有的请求起作用，但是拦截器只能对action请求起作用（比如访问静态文件，过滤器会对所有进入tomcat的请求进行过滤，但是拦截器不会对图片等进行处理，而只能处理aciton）。
- 注入Bean情况不同：拦截器先于ApplicationContext加载，所以拦截器无法注入Spring容器管理的Bean，如果利用自动注入会报空指针
  - 解决办法：拦截器不使用@Component加载，改为使用@Configuration+@Bean加载
- 控制执行顺序不同：过滤器用@Order注解控制执行顺序，通过@Order控制过滤器级别，值越小级别越高，越优先执行；拦截器默认的执行顺序，就是注册顺序。也可以通过Order手动设置控制，值越小越先执行。



#### 过滤器应用：

- 用户授权的Filter：Filter负责检查用户请求，根据请求过滤用户非法请求
- 日志Filter：详细记录某些特殊的用户请求
- 负责解码的Filter：对非标准编码的请求解码
- 改变XML内容的XSLTFilter