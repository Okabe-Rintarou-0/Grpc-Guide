# C++
这是 C++ 使用 grpc 的指南。

## 工具

+ bazel/cmake
+ conan
+ npm
+ protoc

## 推荐资料

+ [Sync and async methods under a service](https://github.com/grpc/grpc/issues/4285)

## 使用说明


### 服务注册

你可以使用 [`grpc::ServerBuilder::RegisterService`]() 来注册一个服务。推荐的做法是将同步和异步的服务分开（即对应两个 proto 文件），然后分别注册。
最好的方式就是一个 proto 中指定的服务都是同步/异步的。当然，能够注册这若干个服务的前提是他们彼此之间都是不同的。

如果你想要在不同的工作负载下跑你的服务，更好的方式是起多个 grpc server，这样保证服务跑在不同的端口，可以根据特定的负载指定特定的配置。

虽然并不推荐在一个服务中混合同步和异步 rpc，但是这个特性确实是可实现的，详见推荐资料。

如果你想要实现 low-level/generic 应用，比如一个 proxy，可以使用 `grpc::ServerBuilder::RegisterAsyncGenericService`。它暴露了更加低层的 api，
并且提供给用户一些 `raw message`.

### 线程模型

grpc 的线程模型需要分为两种情况进行讨论：同步和异步。首先，对于同步而言，每次进行 rpc 都会使用（注意我说的是使用，不一定是创建，因为它可能是从线程池中
获取的空闲的线程）一个对应的 handler thread。我们可以通过以下方式来控制最大的线程数量：

+ 首先，创建一个 `grpc::ResourceQuota` 对象，并且通过 `grpc::ResourceQuota::SetMaxThreads` 方法设置最大线程数量；
+ 最后，通过 `grpc::ServerBuilder::SetSourceQuota` 来应用这个配置。

另外，查阅资料的时候发现了以下描述：
> There is actually just one thread pool for both polling and request handlers. When a request comes in, an existing polling 
> thread basically becomes a request handler thread, and when the request handler completes, that thread is available to become a polling 
> thread again. The `MIN_POLLERS` and `MAX_POLLERS` options allow tuning the number of threads that are used for polling: when a polling
> thread becomes a request handler thread, if there are not enough polling threads remaining, a new one wil be spawned, and when a request    
> handler finishes, if there are too many polling threads, the thread will terminate. But none of this affects the number of threads used for
> request handlers -- that is still unbounded by default.                                                    

也就是说 `MIN_POLLERS` 和 `MAX_POLLERS` 能够在一定程度上限制 handler thread 的数量，但是这种控制并不是完全的。也就是说，如果有一时刻 request 特别多，
并不能保证每一时刻的线程数量都在 `MIN_POLLERS` 和 `MAX_POLLERS` 范围内。这也恰巧说明了，对于高并发请求里的场景，使用异步服务是更好的选择。

上述的两个参数也是可调的，可以通过 `grpc::ServerBuilder::SetSyncServerOption` 进行设置。

对于异步服务而言，推荐使用自己的线程模型来处理。C++ api 文档中提及：

> Right now, the best performance trade-off is having numcpu's threads and one completion queue per thread.

也就是说，当前比较好的方式就是启用和处理器数量相当的线程，并且分配给他们每个线程一个 `Completion queue`.

### Completion Queue
上文中提到了 `Completion Queue`（后文简称 `cq`）。这个数据结构相当于一个消息队列，被用于异步 api 中。 当建立连接，写入数据（读出数据时），都可以将一个指定的 
`tag` 放入到一个 cq 中。这个 `tag` 实际上就是一个 `void *` 类型的对象，可以放一个枚举数值，当然也可以是一个对象的地址。一种比较常见的方式是开启一个线程，不停地调用
`cq.Next` 来获取 `tag` 来异步处理（这个 `tag` 是完成一些任务之后加入到队列的）。前面提到 `tag` 可以是一个对象的地址。于是乎我们可以维护一个类似 `session` 的对象，
假设我们的 rpc 使用的是流式的 response，那么我们会有一个 `grpc::ServerAsyncWriter` 对象来写入数据到流中。我们可以在 `grpc::ServerAsyncWriter::Write` 中指定任务完成后需要放入队列 cq 中的 `tag`，这个 `tag` 可以是一个 `session` 对象的地址。我们可以对 `session` 的状态进行更新和维护。在前面提及的线程获取 `tag` 的时候，可以将其
强制转换为 `session` 指针类型，并且由此获取当前 `session` 的状态，上下文等信息，依次来完成一次异步的 rpc。可能不是那么好理解，在接下来的内容中，会附上一个 example 的网址。

如果你查看通过 `protoc` 生成的代码，你会发现异步 service 中会有一个叫做 `RequestXXX` 的方法。这个 XXX 是你的 rpc 的名字。比如下面的这个例子：
```protobuf
rpc Greet(Request) returns (stream Response) {}
```
那么这个方法就会叫做 `RequestGreet`。

这个方法会使用两个 `cq`，`call_cq` 和 `notification_cq`。经过查阅资料，得知两者的区别：
> Notification_cq gets the tag back indicating a call has started. All subsequent operations(reads, writes, etc) on that call
> report back to call_cq.