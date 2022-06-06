### SpringMVC

* **工作原理及流程**

  - 客户的请求会发送到DispatcherServlet
  - DispatcherServlet在接受到请求后会调用HandlerMapping(处理器映射器)
  - 处理器映射器在找到具体的处理器，会生成处理器对象以及处理器拦截器返回给DispatherServlet
  - DispatcherServlet调用处理器适配器（HandlerAdapter）
  - 处理器适配器（HandlerAdapter）经过适配，调用合适的后端处理器，即Controller
  - HandlerAdapter(处理器适配器)把返回的ModelAndView返回给DispatcherServlet
  - DisPatcherServlet把ModelAndView返回给ViewReslover（视图解析器）
  - ViewReslover返回具体的view
  - DispatcherServlet根据具体的view来进行页面的渲染，最后响应客户；
  - ![图片](https://github.com/havenBoy/havenboy-java-Interview/blob/master/image/1.jpg)

* **涉及到的设计模式**
  - 责任链模式
    - 在DispatcherServlet中的HandlerExecutionChain这个类时具体责任链模式实现的具体类；
    - 它可以把原本耦合的顺序处理的代码和逻辑解耦，因此只需要关心处理的顺序不需要关系具体的处理逻辑，而是转交给具体模块的负责人，简化了请求的处理；
  - 策略模式
    - 在初始化DispatcherServlet组件中，使用getDefaultStrategies方法来决定不同组件的默认类型；
    - 在initLocaleResolver方法，它初始化了一个默认的本地化处理组件，传入的参数是LocaleResolver.class，这个类有很多的实现类，包括AcceptHeaderLocaleResolver、CookieLocaleResolver、FixedLocaleResolver等，从而对应不同的处理方式，增加了代码的可读性性与可维护性；
  - 建造者模式
    - 定义：把一个复杂的对象的构建与表示分离，使得相同的构建方式得到不同的表示；
  - 组合模式
    - 定义：
* **常用注解介绍**
  - @Controller   用于标记一个类上
  - @RequestMapping   用来处理请求地址映射的注解，可用于类或者方法上，以下为属性：
    - value   : 实际的请求地址
    - method  : 指定请求的类型，如GET  POST  DELETE  PUT
    - Consumes : 指定处理请求提交的内容类型，例如application/json    text/html
    - Produces : 指定返回的内容类型；
    - Params :  参数中必须包含某个参数才可以被处理
    - header ： 请求中的header中必须包含特定的内容才可以处理请求；
  - @Autowired ： 做bean的注入
  - @PathVariable :   将请求URL中的模板变量映射到功能处理方法的参数
  - @ResponsBody : 将Controller返回的对象，通过HttpMessageConverter写入到Response对象的Body
  - @ModelAttribte : 
    - 方法上： 在调用请求之前，会逐个调用带有此标签的方法，之后把参数放入映射方法的Map中；
    - 参数中：先从模型数据中获取对象，之后把对象绑定到参数中；
  - @SessionAttributes : 
    - 只能定义在类上；
    - 多个请求可以共用某个模型属性数据；

