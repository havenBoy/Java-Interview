### SpringMVC

* 工作原理及流程
  - 客户的请求会发送到DispatcherServlet
  - DispatcherServlet在接受到请求后会调用HandlerMapping(处理器映射器)
  - 处理器映射器在找到具体的处理器，会生成处理器对象以及处理器拦截器返回给DispatherServlet
  - DispatcherServlet调用处理器适配器（HandlerAdapter）
  - 处理器适配器（HandlerAdapter）经过适配，调用合适的后端处理器，即Controller
  - HandlerAdapter(处理器适配器)把返回的ModelAndView返回给DispatcherServlet
  - DisPatcherServlet把ModelAndView返回给ViewReslover（视图解析器）
  - ViewReslover返回具体的view
  - DispatcherServlet根据具体的view来进行页面的渲染，最后响应客户

