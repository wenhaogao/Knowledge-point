              Spring-MVC概述

大部分java应用都是用Web应用，展示层是Web引用不可忽略的重要环节。Spring为展示层提供了一个优秀的框架———Spring-Mvc。Spring-Mvc在数据绑定，视图解析，本地化处理及静态资源处理上都有着很好的表现。它在框架设计，扩展性，灵活性等方面全面超越了Struts，WebWork等MVC框架。


1.Spring MVC体系结构




2.Spring工作流程描述
1、用户发送请求至前端控制器DispatcherServlet 
2、DispatcherServlet收到请求调用HandlerMapping处理器映射器。 
3、处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(二者组成HandlerExecutionChain),并将其一并返回给DispatcherServlet。 
4、DispatcherServlet通过HandlerAdapter处理器适配器调用处理器 
5、执行处理器(Controller，也叫后端控制器)。 
6、Controller执行完成返回ModelAndView 
7、HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet 
8、DispatcherServlet将ModelAndView传给ViewReslover视图解析器 
9、ViewReslover解析后返回具体View 
10、DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。 
11、DispatcherServlet对用户进行响应

3.DispatcherServlet的作用
  DispatcherServlet是Spring-mvc的灵魂和心脏，它负责HTTP请求并协调Spring-mvc的各个组件完成请求处理的工作。和任何Servlet一样，用户必须在web.xml中配置好DispatcherServlet。

4.注解驱动的控制器
1.使用requestMapping映射请求
在POJO类定义处标注@Controller，在通过<context:component-scan/>扫描相应的类包，即可使POJO成为一个能处理HTTP请求的控制器。通过请求URL进行映射，@RequestMapping使用value的值指定请求的URL，需要注意的是URL相对于Web应用的部署路径，而在方法处和定义处指定的URL则相对于类定义出指定的URL。
2.使用@RequestParam绑定请求参数值
   因为Java类反射对象默认不记录方法入参的名称，因此需要在方法入参处使用@RequestParam注解指定其对应参数的请求。
  *value:参数名
*required:是否为必须的，默认为true，表示请求中必须包含对应的参数名。
*defaultValue:默认参数名，设置该参数时，自动将required设为false。
3.使用@CookieValue绑定请求中的Cookie值
    *使用@CookieValue可让处理方法入参绑定某个Cookie的值，他和@RequestParam拥有3个相同的参数。
4.使用@RequestHeader绑定请求报文的属性值
 *请求报文包含了若干报文头属性，服务器可据此获知客户端的信息，通过@RequestHeader即可将报文头属性值绑定到属性方法中。

5.处理数据模型
 *ModelAndView:处理方法返回值类型为ModelAndView时，方法体即可通过该对象添加模型数据。
*@ModelAttribute:方法入参标注该注解后，入参对象就会放入数据模型中。
*@SessionAttributes:将模型中的某个属性暂存到HttpSession中，以便多个请求之间可以共享这个属性。
*Map及Model:入参为org.springframework.ui.Model，org.springframework.ui.ModelMap或java.uitl.Map时，处理方法返回时，Map中的数据会自动添加到模型中。













   