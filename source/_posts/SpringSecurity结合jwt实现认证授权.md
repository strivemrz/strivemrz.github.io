---
updated: 2022-11-07
created: 2022-11-07
title: SpringSecurity结合jwt实现认证授权
categories: springBoot
tags:
  - springBoot

excerpt: ''

cover: https://ts1.cn.mm.bing.net/th/id/R-C.31c9be0cbc6c50d9274d6fdbb141229f?rik=JN%2f2MlaESDUgjQ&riu=http%3a%2f%2fwww.kutoo8.com%2fupload%2fimage%2f71901020%2f2016030806.jpg&ehk=1E%2bnqytS8AmGASIV0Deerx4S%2btmQSZ0WrTqmoD8ZQvg%3d&risl=&pid=ImgRaw&r=0

---

## 引入SpringSecurity

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 工具类

### jwt工具类

```java
public class JwtUtils {

    /**
     * 两个常量： 过期时间；秘钥
     */
    public static final long EXPIRE = 1000*60*60*24;
    public static final String SECRET = "ukc8BDbRigUDaY6pZFfWus2jZWLPHO";

    /**
     * 生成token字符串的方法
     * @param id
     * @param nickname
     * @return
     */
    public static String getJwtToken(String id,String nickname){
        String JwtToken = Jwts.builder()
                    //JWT头信息
                    .setHeaderParam("typ", "JWT")
                    .setHeaderParam("alg", "HS2256")
                    //设置分类；设置过期时间 一个当前时间，一个加上设置的过期时间常量
                    .setSubject("lin-user")
                    .setIssuedAt(new Date())
                    .setExpiration(new Date(System.currentTimeMillis() + EXPIRE))
                    //设置token主体信息，存储用户信息
                    .claim("id", id)
                    .claim("nickname", nickname)
                    //.signWith(SignatureAlgorithm.ES256, SECRET)
                    .signWith(SignatureAlgorithm.HS256, SECRET)
                    .compact();
        return JwtToken;
    }

    /**
     * 判断token是否存在与有效
     * @Param jwtToken
     */
    public static boolean checkToken(String jwtToken){
        if (StringUtils.isEmpty(jwtToken)){
            return false;
        }
        try{
            //验证token
            Jwts.parser().setSigningKey(SECRET).parseClaimsJws(jwtToken);
        }catch (Exception e){
//            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 判断token是否存在与有效
     * @Param request
     */
    public static boolean checkToken(HttpServletRequest request){
        try {
            String token = request.getHeader("token");
            if (StringUtils.isEmpty(token)){
                return false;
            }
            Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token);
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 根据token获取会员id
     * @Param request
     */
    public static String getMemberIdByJwtToken(HttpServletRequest request){
        String token = request.getHeader("token");
        if (StringUtils.isEmpty(token)){
            return "";
        }
        Jws<Claims> claimsJws = Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token);
        Claims body = claimsJws.getBody();
        return (String) body.get("id");
    }
    /**
     * 根据token字符串获取id
     */
    public static String getIdByToken(String token){
        if(StringUtils.isEmpty(token)){
            return "";
        }
        Jws<Claims> claimsJws = Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token);
        Claims body = claimsJws.getBody();
        return (String) body.get("id");
    }
}
```

### Redis工具类

```java
public class RedisUtil {
   private static String ip="192.168.0.106";
   private static int port=6379;
   private static int timeout=10000;
   private static JedisPool pool=null;
   
   
   static{
      JedisPoolConfig config=new JedisPoolConfig();
      config.setMaxTotal(1024);//最大连接数
      config.setMaxIdle(200);//最大空闲实例数
      config.setMaxWaitMillis(3000);//等连接池给连接的最大时间，毫秒
      config.setTestOnBorrow(true);//borrow一个实例的时候，是否提前vaildate操作

      pool=new JedisPool(config,ip,port,timeout,"123456");
      
   }
   
   //得到redis连接
   public static Jedis getJedis(){
      if(pool!=null){
         return pool.getResource();
      }else{
         return null;
      }
   }
   
   //关闭redis连接
   public static void close(final Jedis redis){
      if(redis != null){
         redis.close();
      }
   }
}
```

### Result工具类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ResultUtils<T> {
    //返回状态码
    private Integer code;
    //返回信息
    private String message;
    //返回数据
    private T data;
}
```

### 序列化工具类

```java
public class SerializeUtil {
   /**
    *
    * 序列化
    */
   public static byte[] serialize(Object obj) {
 
      ObjectOutputStream oos = null;
      ByteArrayOutputStream baos = null;
 
      try {
         // 序列化
         baos = new ByteArrayOutputStream();
         oos = new ObjectOutputStream(baos);
 
         oos.writeObject(obj);
         byte[] byteArray = baos.toByteArray();
         return byteArray;
 
      } catch (IOException e) {
         e.printStackTrace();
      }
      return null;
   }
 
   /**
    *
    * 反序列化
    *
    * @param bytes
    * @return
    */
   public static Object unSerialize(byte[] bytes) {
 
      ByteArrayInputStream bais = null;
 
      try {
         // 反序列化为对象
         bais = new ByteArrayInputStream(bytes);
         ObjectInputStream ois = new ObjectInputStream(bais);
         return ois.readObject();
 
      } catch (Exception e) {
         e.printStackTrace();
      }
      return null;
   }

}
```

### WebUtils工具类

```java
public class WebUtils {
    public static void renderString(HttpServletResponse response,String message) throws IOException {
        response.setStatus(200);
        response.setContentType("application/json");
        response.setCharacterEncoding("utf-8");
        response.getWriter().write(message);
    }
}
```

## 流程图

### SpringSecurity过滤器链

![springsecurity执行流程](/2022/11/07/SpringSecurity结合jwt实现认证授权/springsecurity执行流程.png)

### UsernamePasswordAuthenticationFilter过滤器执行流程

![usernamepassword过滤器流程图](/2022/11/07/SpringSecurity结合jwt实现认证授权/usernamepassword过滤器流程图.png)

### 数据库RABC模式

![RBAC权限模型](/2022/11/07/SpringSecurity结合jwt实现认证授权/RBAC权限模型.png)

## 更改成从数据库查询用户名密码

### 重写UserDetails实体类

```java
@Data
@NoArgsConstructor
public class UserDetail implements UserDetails {
    private Admin admin;
    //权限信息
    private List<String> permissions;

    public UserDetail(Admin admin, List<String> permissions) {
        this.admin = admin;
        this.permissions = permissions;
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        //获取权限方法
        List<SimpleGrantedAuthority> collect = permissions.stream().map(i -> {
            return new SimpleGrantedAuthority(i);
        }).collect(Collectors.toList());
        return collect;
    }
    @Override
    public String getPassword() {
        return admin.getPassword();
    }
    @Override
    public String getUsername() {
        return admin.getUsername();
    }
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

### 重写UserDetailsService方法实现从数据库查询用户名密码

```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private AdminService adminService;
    @Autowired
    private AdminMapper adminMapper;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        QueryWrapper<Admin> queryWrapper=new QueryWrapper<>();
        queryWrapper.eq("username",username);
        Admin admin = adminService.getOne(queryWrapper);
        if(Objects.isNull(admin)){
            throw new RuntimeException("用户名密码错误");
        }
        //TODO 授权操作
//        将权限查询出来，数据库采用RABC方式
        List<String> list = adminMapper.selectPermissions(admin.getId().toString());
        return new UserDetail(admin,list);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.it.mapper.AdminMapper">
<!-- 权限查询 -->
   <select id="selectPermissions" parameterType="java.lang.String" resultType="java.lang.String">
      select
         m.perm_key
      FROM
         admin_role ar left JOIN admin on ar.user_id=admin.id
                    LEFT JOIN role r on ar.role_id =r.id
                    LEFT JOIN role_menu rm on rm.role_id = r.id
                    LEFT JOIN menu m on m.id = rm.menu_id
      where admin.id=#{value};
   </select>
</mapper>
```

### 自定义数据库加密方式/注册AuthenticationManager类/SpringSecurity相关配置

```java
@Configuration
public class Bcrypt extends WebSecurityConfigurerAdapter {
    @Autowired
    private JWTAuthenticationTokenFilter jwtAuthenticationTokenFilter;
    @Autowired
    private AuthenticationEntryPoint authenticationEntryPoint;//自定义认证异常处理类
    @Autowired
    private AccessDeniedHandler accessDeniedHandler;//自定义权限异常处理类

    @Bean
    public PasswordEncoder Passencoder(){
        //数据加密方式
        return new BCryptPasswordEncoder();
    }

    //用户认证bean
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
            //关闭csrf
            csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                //其他的配置
                .and()
                //对认证请求做相应设置
                .authorizeRequests()
                //对于登录接口 允许匿名访问 登录了就不能访问了，permitAll匿名和登录都能访问
                .antMatchers("/user/login").anonymous()
                //除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
        //添加jwt认证过滤器
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        //定义认证异常处理类
        http.exceptionHandling()
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler);
    }
}
```

## 登录/登出

### controller层

```java
@PostMapping("/user/login")
public Object login(Admin admin){
    ResultUtils login = adminService.login(admin);
    return login;
}
```

### service层

```java
@Service
public class AdminServiceImpl extends ServiceImpl<AdminMapper, Admin> implements AdminService {

    @Autowired
    private AdminMapper adminMapper;
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public ResultUtils login(Admin admin) {
        //用户认证
        UsernamePasswordAuthenticationToken authenticationToken=new UsernamePasswordAuthenticationToken(admin.getUsername(),admin.getPassword());
        Authentication authenticate = authenticationManager.authenticate(authenticationToken);
        if(Objects.isNull(authenticate)){
            //如果用户名密码错误，或者没有权限则返回null
            throw new RuntimeException("认证异常");
        }
        UserDetail userDetail = (UserDetail)authenticate.getPrincipal();
        Admin admin1 = userDetail.getAdmin();
        String id=admin1.getId().toString();
        String username = admin1.getUsername();
        //认证成功生成token
        String jwtToken = JwtUtils.getJwtToken(id, username);
        //将带有用户信息加入到redis缓存
        Jedis jedis = RedisUtil.getJedis();
        jedis.set(("token:"+id).getBytes(), SerializeUtil.serialize(userDetail));
        //返回用户信息
        Map<String,String> map=new HashMap<>();
        map.put("token",jwtToken);
        return new ResultUtils<Map<String,String>>(200, "认证成功", map);
    }

    @Override
    public ResultUtils logout() {
        //登出
        //获取SecurityContextColder中的数据id
        UsernamePasswordAuthenticationToken authentication = (UsernamePasswordAuthenticationToken)SecurityContextHolder.getContext().getAuthentication();
        Admin principal = (Admin)authentication.getPrincipal();
        //将redis中的数据删除
//        if(Objects.isNull(principal)){
//            //SecurityContextColder中没有用户信息
//        }
        Jedis jedis = RedisUtil.getJedis();
        Long del = jedis.del("token:" + principal.getId());
        return new ResultUtils<Long>(200,"登出成功",del);
    }
}
```

## token认证过滤器

```java
@Component
@Slf4j
public class JWTAuthenticationTokenFilter extends OncePerRequestFilter {

    /**
     * todo 添加认证过滤器
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String token = request.getHeader("token");
//        System.out.println(!StringUtils.hasText(token)+" "+JwtUtils.checkToken(token));
        if((!StringUtils.hasText(token))||(!JwtUtils.checkToken(token))){
            //如果未携带token,或者token过期，则直接放行
            filterChain.doFilter(request,response);
            return;
        }
        //解析token
        String idByToken = JwtUtils.getIdByToken(token);
        byte[] adminByt = RedisUtil.getJedis().get(("token:" + idByToken).getBytes());
        if(adminByt==null||adminByt.length==0){
            log.warn("用户已登出");
            filterChain.doFilter(request,response);
            return;
        }
        UserDetail userDetail = (UserDetail)SerializeUtil.unSerialize(adminByt);
//        设置该用户为已认证
        UsernamePasswordAuthenticationToken authenticationToken=new UsernamePasswordAuthenticationToken(userDetail,null,userDetail.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        filterChain.doFilter(request,response);
    }
}
```

## 开启权限

```java
@SpringBootApplication
@MapperScan("com.it.mapper")
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SpringbootLinster {
    public static void main(String[] args) {
//        SpringApplication.run(SpringbootLinster.class,args);
        SpringApplication application = new SpringApplication(SpringbootLinster.class);
        application.addListeners(new FirstLinster());
        ConfigurableApplicationContext run = application.run(args);
    }
}
```

```java
@GetMapping("/hello")
@PreAuthorize("hasAuthority('sys:book:delete')")
public String showIndex(){
    return "hello";
}
```

## 自定义认证、权限异常处理

### 认证异常处理

```java
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    //自定义授权异常处理类
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
        ResultUtils<String> resultUtils=new ResultUtils<>(HttpStatus.FORBIDDEN.value(),"没有该权限",null);
        String resultMessage = JSON.toJSONString(resultUtils);
        WebUtils.renderString(httpServletResponse,resultMessage);
    }
}
```

### 权限异常处理

```java
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    //自定义认证异常处理类
    @Override
    public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        ResultUtils<String> resultUtils=new ResultUtils(401,"用户名密码错误",null);
        String resultMess = JSON.toJSONString(resultUtils);
        WebUtils.renderString(httpServletResponse,resultMess);
    }
}
```

### 在SpringSecurity配置中设置异常处理类

```java
//定义认证异常处理类
http.exceptionHandling()
        .authenticationEntryPoint(authenticationEntryPoint)
        .accessDeniedHandler(accessDeniedHandler);
```