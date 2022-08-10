### Filter

- Filter称为过滤器，在WEB开发中用于对所有的请求进行拦截
- *Java*中的*Filter* 并不是一个标准的*Servlet* ，它不能处理用户请求，也不能对客户端生成响应。 主要用于对*HttpServletRequest* 进行预处理，也可以对*HttpServletResponse* 进行后处理，是个典型的处理链。 



#### Filter的生命周期

```java
public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {
    }

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    default void destroy() {
    }
}
```

- 过滤器Filter也具有生命周期：init()->doFilter()->destroy()，由部署文件中的filter元素驱动。在servlet2.4中，过滤器同样可以用于请求分派器，但须在web.xml中声明，<dispatcher>INCLUDE或FORWARD或REQUEST或ERROR</dispatcher>该元素位于filter-mapping中。 



#### Filter的过滤器链



- 在一个web应用中，可以开发编写多个Filter，这些Filter组合起来称之为一个Filter链。web服务器根据Filter在web.xml文件中的注册顺序，决定先调用哪个Filter，当第一个Filter的doFilter方法被调用时，web服务器会创建一个代表Filter链的FilterChain对象传递给该方法。在doFilter方法中，开发人员如果调用了FilterChain对象的doFilter方法，则web服务器会检查FilterChain对象中是否还有filter，如果有，则调用第2个filter，如果没有，则调用目标资源。