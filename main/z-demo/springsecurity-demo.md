# Spring Security Demo

## 前言

网上用的比较多的方式是UserDetailsService方案，这种方案难以实现多种登录方式。本文用Filter+Manager+Provider+Token的方案实现多种登录方式。代码过多，本文只展示其中一种登录方式。 简单描述下这几个东西的作用：

- Filter负责拦截请求并调用Manager
- Manager负责管理多个Provider，并选择合适的Provider进行认证
- Provider负责认证，检查账号密码之类的
- Token是认证信息，包含账号密码，具体可以自己定

## 实现

### Filter

```java
public class UserPasswordAuthenticationProcessingFilter extends AbstractAuthenticationProcessingFilter {
    protected UserPasswordAuthenticationProcessingFilter() {
        super("/login");//认证url
    }


    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse httpServletResponse) throws AuthenticationException, IOException, ServletException {
        //根据前端参数判断是哪种登录类型，封装成对应方式的Token，提交给Manager
        if("password".equals(request.getParameter("type"))){
            String user = request.getParameter("username");
            String password = request.getParameter("password");
            System.out.println(user);
            //把账号密码封装成token，传给manager认证
            return getAuthenticationManager().authenticate(new UserPasswordAuthenticationToken(user,password));
        }else{
            //其他登录方式
            return null;
        }
    }
}

```





## Reference

- https://juejin.cn/post/6914959662529904653