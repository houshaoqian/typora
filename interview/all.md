## Spring
### Spring
### SpringMVC
1. SpringMVC流程
	1.前端控制器DispatcherServlet#doDispatch接受任何url请求；
	2.前端控制器通过handlerMappings匹配得到一个调用链HandlerExecutionChain，调用链中保存的是拦截器Intercepter，控制器Controller等；
	3.通过调用链获取适配器HandlerAdapter，适配器也成为后端控制器，负责执行具体的Controller和返回数据视图ModelAndVIew对象给前端控制器；
	4.前端控制器将数据视图交给视图解析器ViewReslover进行渲染处理，处理完成后返回视图View给前端控制器；
	5.前端控制器将视图通过相应Response相应给用户。
2. 