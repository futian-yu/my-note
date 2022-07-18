## 使用@ControllerAdvice + @ExceptionHandler 注解实现Controller层异常全局处理


**0、背景：**
分享下项目中使用的两个十分有效的注解，用于对Controller层异常实现全局统一处理，十分nice！

### 1、前言

对于与数据库相关的 Spring MVC 项目，我们通常会把事务配置在 Service层，当数据库操作失败时让 Service 层抛出运行时异常，Spring 事物管理器就会进行回滚。

如此一来，我们的 Controller 层就不得不进行 try-catch Service 层的异常，否则会返回一些不友好的错误信息到客户端。但是，Controller 层每个方法体都写一些模板化的 try-catch 的代码，很难看也难维护，特别是还需要对 Service 层的不同异常进行不同处理的时候。

所以我们可以使用 @ControllerAdvice + @ExceptionHandler 注解进行全局的 Controller 层异常处理，只要设计得当，就再也不用在 Controller 层进行 try-catch 了。

### 2、介绍

@ExceptionHandler：统一处理某一类异常，从而能够减少代码重复率和复杂度
@ControllerAdvice：异常集中处理，更好的使业务逻辑与异常处理剥离开
@ResponseStatus：可以将某种异常映射为HTTP状态码

@ExceptionHandler：
当一个Controller中有方法加了@ExceptionHandler之后，这个Controller其他方法中没有捕获的异常就会以参数的形式传入加了@ExceptionHandler注解的那个方法中。

@ControllerAdvice：
该注解作用对象为TYPE，包括类、接口和枚举等，在运行时有效，并且可以通过Spring扫描为bean组件。其可以包含由@ExceptionHandler、@InitBinder 和@ModelAttribute标注的方法，可以处理多个Controller类，这样所有控制器的异常可以在一个地方进行处理。

注：如果单使用@ExceptionHandler，只能在当前Controller中处理异常。但当配合@ControllerAdvice一起使用的时候，就可以摆脱那个限制，对所有controller层异常进行处理。

### 3、实际项目演示

```java
/**

- 全局的的异常拦截器（拦截所有的控制器）（带有@RequestMapping注解的方法上都会拦截）
  *

- @author yj

- @date 2018年05月22日 下午3:19:56
  */
  @ControllerAdvice
  public class GlobalExceptionHandler {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  /**

  - 拦截业务异常
    */
    @ExceptionHandler(BussinessException.class)   //此处为自定义业务异常类
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)   //返回一个指定的http response状态码
    @ResponseBody
    public ErrorTip notFount(BussinessException e) {
    LogManager.me().executeLog(LogTaskFactory.exceptionLog(ShiroKit.getUser().getId(), e));
    getRequest().setAttribute("tip", e.getMessage());
    log.error("业务异常:", e);
    return new ErrorTip(e.getCode(), e.getMessage());
    }

  /**

  - 用户未登录
    */
    @ExceptionHandler(AuthenticationException.class)   //此处为shiro未登录异常类
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public String unAuth(AuthenticationException e) {
    log.error("用户未登陆：", e);
    return "/login.html";
    }

  /**

  - 账号被冻结
    */
    @ExceptionHandler(DisabledAccountException.class)   //此处为shiro账号冻结异常类
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public String accountLocked(DisabledAccountException e, Model model) {
    String username = getRequest().getParameter("username");
    LogManager.me().executeLog(LogTaskFactory.loginLog(username, "账号被冻结", getIp()));
    model.addAttribute("tips", "账号被冻结");
    return "/login.html";
    }

  /**

  - 账号密码错误
    */
    @ExceptionHandler(CredentialsException.class)   //此处为shiro账号密码错误异常类
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    public String credentials(CredentialsException e, Model model) {
    String username = getRequest().getParameter("username");
    LogManager.me().executeLog(LogTaskFactory.loginLog(username, "账号密码错误", getIp()));
    model.addAttribute("tips", "账号密码错误");
    return "/login.html";
    }

  /**

  - 验证码错误
    */
    @ExceptionHandler(InvalidKaptchaException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public String credentials(InvalidKaptchaException e, Model model) {
    String username = getRequest().getParameter("username");
    LogManager.me().executeLog(LogTaskFactory.loginLog(username, "验证码错误", getIp()));
    model.addAttribute("tips", "验证码错误");
    return "/login.html";
    }

  /**

  - 无权访问该资源
    */
    @ExceptionHandler(UndeclaredThrowableException.class)
    @ResponseStatus(HttpStatus.UNAUTHORIZED)
    @ResponseBody
    public ErrorTip credentials(UndeclaredThrowableException e) {
    getRequest().setAttribute("tip", "权限异常");
    log.error("权限异常!", e);
    return new ErrorTip(BizExceptionEnum.NO_PERMITION);
    }

  /**

  - 拦截未知的运行时异常
    */
    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ResponseBody
    public ErrorTip notFount(RuntimeException e) {
    LogManager.me().executeLog(LogTaskFactory.exceptionLog(ShiroKit.getUser().getId(), e));
    getRequest().setAttribute("tip", "服务器未知运行时异常");
    log.error("运行时异常:", e);
    return new ErrorTip(BizExceptionEnum.SERVER_ERROR);
    }

  /**

  - session失效的异常拦截
    */
    @ExceptionHandler(InvalidSessionException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public String sessionTimeout(InvalidSessionException e, Model model, HttpServletRequest request, HttpServletResponse response) {
    model.addAttribute("tips", "session超时");
    assertAjax(request, response);
    return "/login.html";
    }

  /**

  - session异常
    */
    @ExceptionHandler(UnknownSessionException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public String sessionTimeout(UnknownSessionException e, Model model, HttpServletRequest request, HttpServletResponse response) {
    model.addAttribute("tips", "session超时");
    assertAjax(request, response);
    return "/login.html";
    }

  private void assertAjax(HttpServletRequest request, HttpServletResponse response) {
      if (request.getHeader("x-requested-with") != null
              && request.getHeader("x-requested-with").equalsIgnoreCase("XMLHttpRequest")) {
          //如果是ajax请求响应头会有，x-requested-with
          response.setHeader("sessionstatus", "timeout");//在响应头设置session状态
      }
  }

}

```

### 4、参考文档

​	https://blog.csdn.net/Abysscarry/article/details/80400140