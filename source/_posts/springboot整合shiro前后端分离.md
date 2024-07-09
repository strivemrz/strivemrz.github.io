---
updated: 2023-04-16
created: 2023-04-16
title: springboot整合shiro前后端分离
categories: springBoot
tags:
  - springBoot

excerpt: ''

cover: https://pic4.zhimg.com/v2-b730c73ebd78bd5f22293aab0d343f4b_r.jpg?source=1940ef5c

---

#### springboot+shiro+jwt前后端分离

#### 导入依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!--mapStruct依赖 高性能对象映射-->
<!--mapstruct核心-->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.0.Beta1</version>
</dependency>
<!--mapstruct编译-->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.0.Beta1</version>
</dependency>
<!--引入shrio-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-starter</artifactId>
    <version>1.5.3</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!--整合druid数据源-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.8</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.2</version>
</dependency>
<!-- 配置使用redis启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--集成jwt实现token认证-->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.2.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
~~~

#### 流程图

![流程图.png](/2023/04/16/springboot整合shiro前后端分离/流程图.png)

#### 数据库设计

##### 数据库采用RBAC模式，即用户->角色->权限表

#### 工具类

##### Result工具类

~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Result<T> {
    /*响应码*/
    private int code;
    /*响应消息*/
    private String msg;
    private T data;
    public Result(T data) {
        this.data = data;
    }

    public Result(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    public static <T> Result<T> success(T data) {
        return new Result<T>(data);
    }
    public static <T> Result<T> error(int code, String msg) {
        return new Result<T>(code, msg);
    }

    public static Result<String> error(String msg){
        return new Result<>(msg);
    }
}
~~~

#### 配置类

##### 自定义Realm数据源

```java
public class UserRealm extends AuthorizingRealm {
    @Autowired
    private AdminService adminService;
    @Autowired
    private StringRedisTemplate redisTemplate;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //授权
        Admin admin = (Admin)principalCollection.getPrimaryPrincipal();//当涉及到权限校验时，走此方法，这里拿的是认证时候设置的Principal
        //此方法需要一个AuthorizationInfo类型的返回值，因此返回一个他的实现类，SimpleAuthorizationInfo
        //通过SimpleAuthorizationInfo里的addStringPermission()设置用户的权限
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //todo 查权限 用户->角色->权限
        simpleAuthorizationInfo.addStringPermission("user:select:*");//简单写了一个，从数据库搜角色权限
        return simpleAuthorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token){
        //认证
        String accessToken=(String)token.getPrincipal();//自定义AuthToken，采用token形式
        //redis 缓存中存值 key为token，value为username
        //根据token去缓存里查找用户名
        String username = redisTemplate.opsForValue().get(accessToken);
        if(username==null){
            throw new IncorrectCredentialsException("token失效，请重新登录");//设置过期时间
        }
        Admin admin=adminService.findByUsername(username);
        if(admin==null){
            throw new UnknownAccountException("用户不存在");
        }
        //此方法需要返回一个AuthenticationInfo类型的数据
        //因此返回一个他的实现类，SimpleAuthenticationInfo，将user以及获取到的token传入他可以实现自动认证，到这代表认证成功
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(admin, accessToken, this.getName());
        return simpleAuthenticationInfo;
    }
}
```

##### 自定义AuthToken

```java
public class AuthToken extends UsernamePasswordToken {
    //前后端分离是无状态的，这里设置获取token而不是username，password
    private String token;

    public AuthToken(String token) {
        this.token = token;
    }
    @Override
    public Object getPrincipal() {
        return token;
    }
    @Override
    public Object getCredentials() {
        return token;
    }
}
```

##### shiroConfig

```java
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(manager());

//        //跳转登录 未认证时跳转
//        bean.setLoginUrl("/admin/toLogin");
//        //未授权时跳转
//        bean.setUnauthorizedUrl("/admin/unAuth");

        //添加shiro的内置过滤器
        /**
         * anon:无需认证就可以访问
         * authc:必须认证才能访问
         * user:必须拥有记住我功能才能用
         * perms:拥有对某个资源的权限才能访问
         * role:拥有某个角色权限才能访问
         */
        //添加过滤器
        Map<String, Filter> filterMap=new HashMap<>();
        //设置自定义过滤器
        filterMap.put("authc",new AuthFilter());
        bean.setFilters(filterMap);

        HashMap<String, String> map = new HashMap<>();
        map.put("/logout","anon");
        map.put("/admin/login","anon");
        map.put("/admin/toLogin","anon");
        map.put("/admin/register","anon");
        map.put("/**","authc");

        bean.setFilterChainDefinitionMap(map);
        return bean;
    }

    //配置seccurityManager的实现类，变相的配置了securityManager
    @Bean
    public DefaultWebSecurityManager manager(){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();

        //关闭shiro自带的session，无状态
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        defaultWebSecurityManager.setSubjectDAO(subjectDAO);

        //设置数据源
        defaultWebSecurityManager.setRealm(userRealm());
        return defaultWebSecurityManager;
    }

    @Bean
    public UserRealm userRealm(){
        UserRealm userRealm = new UserRealm();
//        userRealm.setCredentialsMatcher(hashedCredentialsMatcher());
        return userRealm;
    }
//    @Bean
//    public HashedCredentialsMatcher hashedCredentialsMatcher(){
//        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
//        //设置加密算法
//        hashedCredentialsMatcher.setHashAlgorithmName("md5");
//        //设置加密次数
//        hashedCredentialsMatcher.setHashIterations(5);
//        return hashedCredentialsMatcher;
//    }

    //通过调用Initializable.init()和Destroyable.destroy()方法,从而去管理shiro bean生命周期
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    //开启shiro权限注解
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(DefaultWebSecurityManager defaultWebSecurityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(defaultWebSecurityManager);
        return advisor;
    }
}
```

##### 自定义过滤器

> 1、用户发起请求首先进入isAccessAllowed（）方法，拦截除了option请求 以外的请求， 浏览器机制就是：在发post、get请求之前，首先会发个option请求进行试探，所以要放行option请求通过，否则你的get、post请求永远进不来。
>
> 2、token不为空的情况下则生成属于自己的token，即创建一个类使其继承  UsernamePasswordToken此类
>
> 3、然后进入onAccessDenied（）方法去校验token是否存在，并执行executeLogin(request,response)方法
>
> 4、全局异常处理是controller层的，过滤器先执行，拦截不到

```java
public class AuthFilter extends AuthenticatingFilter {

    /**
     * 生成自定义token 这里重写了executeLogin 方法没用
     */
    @Override
    protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest httpServletRequest=(HttpServletRequest)servletRequest;
        String token = httpServletRequest.getHeader("token");
        return new AuthToken(token);
    }

    /**
     * 所有请求全部拒绝访问
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        if(((HttpServletRequest)request).getMethod().equals(RequestMethod.OPTIONS.name())){
            return true;
        }
        return false;
    }

    /**
     * 拒绝访问的请求
     */
    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request=(HttpServletRequest)servletRequest;
        HttpServletResponse response=(HttpServletResponse)servletResponse;
        String token = request.getHeader("token");
        if(StringUtils.isEmpty(token)){
            String msg= URLEncoder.encode("请先登录","utf-8");
            response.sendRedirect("/exception/"+400+"/"+msg);
            return false;
        }
        try {
            executeLogin(servletRequest,servletResponse);
        }catch (Exception e){
            String msg= URLEncoder.encode("token验证错误","utf-8");
            response.sendRedirect("/exception/"+400+"/"+msg);
            return false;
        }

        return true;
    }
    /**
     * 重写AuthenticatingFilter的executeLogin方法丶执行登陆操作
     */
    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String token = httpServletRequest.getHeader("token");//获取token

        AuthToken jwtToken = new AuthToken(token);
        // 提交给realm进行登入,如果错误他会抛出异常并被捕获, 反之则代表登入成功,返回true
        getSubject(request, response).login(jwtToken);
        return true;
    }

    /**
     * token失效时调用
     */
    @SneakyThrows
    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request, ServletResponse response) {
        HttpServletRequest httpServletRequest= (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        String msg= URLEncoder.encode("登陆凭着已失效,请重新登录","utf-8");
        httpResponse.sendRedirect("/exception/"+400+"/"+msg);
        return false;
    }
}
```

##### 枚举

```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public enum BaseEnum {
    MD5("0","MD5");
    private String code;
    private String value;
    public static String getValueByCode(String code){
        for (BaseEnum baseEnum:values()){
            if(baseEnum.getCode().equals(code)){
                return baseEnum.getValue();
            }
        }
        return null;
    }
}
```

#### controller

```java
@RestController
public class IndexController {
    @Autowired
    private AdminService adminService;
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/admin/login")
    public Result<String> login(@RequestBody Admin admin){
        if(Objects.isNull(admin)){
            return Result.error("请求错误");
        }
        Admin admin1 = adminService.findByUsername(admin.getUsername());
        if(admin1==null){
            return Result.error("账号不存在");
        }else if(!admin.getPassword().equals(admin1.getPassword())){
           	//todo 这里需要对密码进行md5加密 hash5次之后再比对
            //MD5特点是 每次加密条件一样生成的加密字符串是相等的(密码，盐，hash次数)
            return Result.error("密码错误");
        }else{
            String token = UUID.randomUUID().toString().replaceAll("-", "");
            stringRedisTemplate.opsForValue().set(token,admin.getUsername(),3600, TimeUnit.SECONDS);//token过期时间一个小时
            Map<String,String> map=new HashMap<>();
            map.put("token",token);
            return Result.success(map.toString());
        }
    }
    @PostMapping("/admin/register")
    public Result<String> register(@RequestBody Admin admin){
        if(!StringUtils.hasText(admin.getUsername())||!StringUtils.hasText(admin.getPassword())){
            return Result.error("传参有误");
        }
        LambdaQueryWrapper<Admin> select = new LambdaQueryWrapper<Admin>()
                .eq(StringUtils.hasText(admin.getUsername()), Admin::getUsername, admin.getUsername());
        Long aLong = adminService.getBaseMapper().selectCount(select);
        if (aLong>0){
            return Result.error("用户已存在");
        }
        SimpleHash simpleHash = new SimpleHash(BaseEnum.MD5.getValue(), admin.getPassword(), null, 5);//hash5次
        admin.setPassword(simpleHash.toString());
        boolean result = adminService.save(admin);
        return Result.success("保存成功");
    }
    @RequestMapping("/test")
    public Result<String> test(){
        return Result.success("测试接口");
    }
    @RequestMapping("/test2")
    @RequiresPermissions("user:select:*")//这里后端写死了，正常从数据库查询权限
    public Result<String> test2(){
        return Result.success("测试接口权限");
    }

    @PostMapping("/logout")
    public Result<String> logout(@RequestHeader("token")String token){
        stringRedisTemplate.delete(token);
        return Result.success("注销成功");
    }
    @RequestMapping("/exception/{code}/{msg}")
    public Result exceptionHandler(@PathVariable("code") String code,@PathVariable("msg") String msg){
        return Result.error(Integer.valueOf(code),msg);
    }
}
```