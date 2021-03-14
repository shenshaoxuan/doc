

# Grpc

## protobuf

####  特点

* 体积小
* 序列化速度快
* 传输速度快

* 定义数据类型

  * 每个字段都有一个类型和名称

* 指定字段编号

  * 您应该为经常出现的消息元素保留字段号1到15。

* 指定字段规则 

  * required:必须恰好具有其中一个字段。
  * optional:可以有0个或其中一个字段(但不能超过一个)
  * repeated:可以重复任意次数(包括0)。重复值的顺序将被保留。

* 添加注释

  * //

* 时间戳

  * import "google/protobuf/timestamp.proto";
  * 语法版本声明必须放在第一行
  * 如果导入不使用也会编译不成功

  

### 定义服务

*  定义proto文件，
  * service名称
  * 请求参数，返回参数，数据类型定义
  * 服务方法定义

* 编译proto文件，生成py文件
  * py：request和response对象
  * grpc.py:与rpc交互的对象，客户端和服务端



### 定义server端

* 实现在proto中定义的方法，继承grpc已经生成的service，覆盖方法即可，并返回定义好的响应对象
* 启动server，将刚定义好的service类，add后启动



### client端

* 配置服务端地址端口，生成channel
* 使用grpc生成的客户端类生成一个请求对象
* 调用对应的方法
* 必须与定义的参数类型，名称一致，否则无法发送请求
* 





#### 请求流程

* 拦截器，这里会首先通过environ(类似flask的server)匹配到用户请求的函数，类似自己写的身份验证装饰器，拦截器第一个参数是原先处理业务的函数，通过函数执行(request, context)

  

* 处理请求_handle_call(rpc_event, generic_handlers, interceptor_pipeline, thread_pool,
                   concurrency_exceeded)
  * 参数
    * generic_handlers，保存着所有handle的映射
  * 流程
    * 根据请求匹配处理函数_find_method_handler，得到method_handler
    * 然后处理_handle_with_method_handler

* 根据请求匹配处理函数_find_method_handler(rpc_event, generic_handlers, interceptor_pipeline)
  * 参数
    * rpc_event，可查询出rpc_event.call_details.method所请求的api。例如test.hello(哪个包下的哪个service下的哪个函数)
    * generic_handlers，保存着所有handle的映射
  * 流程
    * 查询处理方query_handlers(handler_call_details)
    * 循环所有的映射，查找出处理方法

* 预处理_handle_with_method_handler(rpc_event, method_handler, thread_pool)
  * 流程
    * 判断传输方式，例如流传输，或者一元到一元处理方式
    * 生成一个state对象
    * 调用_handle_unary_unary
* 处理_handle_unary_unary(rpc_event, state, method_handler, default_thread_pool)
  * 流程
    * 通过线程池，去处理响应

* server文件中的_unary_response_in_pool(rpc_event, state, behavior, argument_thunk,
                              request_deserializer, response_serializer)
  * 参数
    * argument_thunk，执行这个函数可生成request对象
  * 流程
    * 调用call_behavior生成response和处理结果
    * 调用_serialize_response序列化response
    * 处理state
    * 销毁上下文
* _server文件中的_call_behavior(rpc_event, state, behavior, argument, request_deserializer)
  * 参数
    * rpc_event:存在着元数据
    * behavior:处理函数
    * argument：请求参数，就是request对象
    * request_deserializer：反序列化函数
  * 流程：
    * 首先根据事件和参数，生成一个上下文
    * 没有生成响应回调函数的话，就生成response，否则要执行回调函数







* ```
  查询收货差异单--冷链
  get:"/api/v2/supply/receiving/diff/cold
  ```

* ```
  根据id查询收货差异单详情
  get:"/api/v2/supply/receiving/diff/{id}"
  ```

* ```
  根据id查商品详情
  get:"/api/v2/supply/receiving/diff/{id}/product"
  ```

* ```
  更新收货单差异单
  put:"/api/v2/supply/receiving/diff/{id}/update"
  ```

