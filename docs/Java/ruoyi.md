# ruoyi的认证过程

> 后端结构

```text
com.ruoyi     
├── common            // 工具类
│       └── annotation                    // 自定义注解
│       └── config                        // 全局配置
│       └── constant                      // 通用常量
│       └── core                          // 核心控制
│       └── enums                         // 通用枚举
│       └── exception                     // 通用异常
│       └── filter                        // 过滤器处理
│       └── utils                         // 通用类处理
├── framework         // 框架核心
│       └── aspectj                       // 注解实现
│       └── config                        // 系统配置
│       └── datasource                    // 数据权限
│       └── interceptor                   // 拦截器
│       └── manager                       // 异步处理
│       └── security                      // 权限控制
│       └── web                           // 前端控制
├── ruoyi-generator   // 代码生成（可移除）
├── ruoyi-quartz      // 定时任务（可移除）
├── ruoyi-system      // 系统代码
├── ruoyi-admin       // 后台服务
├── ruoyi-xxxxxx      // 其他模块
```

ruoyi-system   系统相关的数据库表操作等，以及数据库处理业务，

ruoyi-admin	后台请求控制

> 前端结构

```text
├── build                      // 构建相关  
├── bin                        // 执行脚本
├── public                     // 公共文件
│   ├── favicon.ico            // favicon图标
│   └── index.html             // html模板
│   └── robots.txt             // 反爬虫
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 主题 字体等静态资源
│   ├── components             // 全局公用组件
│   ├── directive              // 全局指令
│   ├── layout                 // 布局
│   ├── router                 // 路由
│   ├── store                  // 全局 store管理
│   ├── utils                  // 全局公用方法
│   ├── views                  // view
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   ├── permission.js          // 权限管理
│   └── settings.js            // 系统配置
├── .editorconfig              // 编码格式
├── .env.development           // 开发环境配置
├── .env.production            // 生产环境配置
├── .env.staging               // 测试环境配置
├── .eslintignore              // 忽略语法检查
├── .eslintrc.js               // eslint 配置项
├── .gitignore                 // git 忽略项
├── babel.config.js            // babel.config.js
├── package.json               // package.json
└── vue.config.js              // vue.config.js
```

> 登录

1、先访问/login控制器	

```java
@PostMapping("/login")
    public AjaxResult login(@RequestBody LoginBody loginBody)
    {
        AjaxResult ajax = AjaxResult.success();
        // 生成令牌
        String token = loginService.login(loginBody.getUsername(), loginBody.getPassword(), loginBody.getCode(),
                loginBody.getUuid());
        ajax.put(Constants.TOKEN, token);
        return ajax;
    }
```

2、loginsevice.login先进行用户验证，loginuser继承implements UserDetails

```
 // 该方法会去调用UserDetailsServiceImpl.loadUserByUsername
            authentication = authenticationManager
                    .authenticate(new UsernamePasswordAuthenticationToken(username, password));
```

认证成功authentication.getPrincipal();获得认证用户

3、生成jwt  tokenService.createToken(loginUser)

```java
public void refreshToken(LoginUser loginUser)
{
    loginUser.setLoginTime(System.currentTimeMillis());
    loginUser.setExpireTime(loginUser.getLoginTime() + expireTime * MILLIS_MINUTE);
    // 根据uuid将loginUser缓存
    String userKey = getTokenKey(loginUser.getToken());
    redisCache.setCacheObject(userKey, loginUser, expireTime, TimeUnit.MINUTES);
}
```

Constants.LOGIN_TOKEN_KEY + uuid设置这个作为redis的key

```java
Map<String, Object> claims = new HashMap<>();
claims.put(Constants.LOGIN_USER_KEY, token);
```

将这个map作为claims传递给create方法

## 载荷（Payload）

 Payload里面是Token的具体内容，也就是Token的数据声明（Claim），这些内容里面有一些是标准字段，也可以添加其他需要添加的内容，如userId,email等。

```java
private String createToken(Map<String, Object> claims)
{
    String token = Jwts.builder()
            .setClaims(claims)//也就是Token的数据声明（Claim）
            .signWith(SignatureAlgorithm.HS512, secret).compact();
    return token;
}
```

## jwt过滤器

JwtAuthenticationTokenFilter

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
        throws ServletException, IOException
{
    LoginUser loginUser = tokenService.getLoginUser(request);
    if (StringUtils.isNotNull(loginUser) && StringUtils.isNull(SecurityUtils.getAuthentication()))
    {
        tokenService.verifyToken(loginUser);
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities());
        authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
    }
    chain.doFilter(request, response);
}
```

tokenService.getLoginUser(request);会获取响应头的

Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJsb2dpbl91c2VyX2tleSI6Ijc3ZWMyMzM5LWVmNDgtNDJjYy1hYjg3LTNjMWUzYTg3ODNkNSJ9.kE7_RADF0Bcn2ovgRmDFXCbFQ2m7BeQ0kjol7-SHc1VPaVtPLwJ0dHrXBEq2AmIJZWVzXFVXxQSsiQbOoCFh-g