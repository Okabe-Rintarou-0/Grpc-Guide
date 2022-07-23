# React

这是 React 使用 grpc 的指南。

## 工具

+ npm
+ protoc
+ ts-protoc-gen
    
    下载方式请见[使用说明](#使用说明)
    
+ protoc-gen-grpc-web
    
    由 google 维护：https://github.com/grpc/grpc-web/releases/tag/1.3.1
    
## 推荐资料
+ [ReactRPC](https://github.com/oslabs-beta/ReactRPC)

+ [@improbable-eng/grpc-web](https://github.com/improbable-eng/grpc-web)

+ [@grpc/grpc-web](https://github.com/grpc/grpc-web)
    
## 使用说明
    
如果要在 React 这样的 web app 中使用 grpc 会比 nodeJs 更加麻烦一点。首先，不能直接使用 @grpc/grpc-js ，也就是 nodeJs 的那套东西。具体的
可以了解一下这个 [issue](https://github.com/grpc/grpc-node/issues/1638)。如果要使用 grpc ，我们必须使用 `grpc-web`。在上面的推荐资料中，
ReactRPC 提供了详细的使用说明。一共有两个 `grpc-web` 的库是可用的，一是 google 维护的 `@grpc/grpc-web`，另一个则是 `@improbable-eng/grpc-web`。
两者都需要一个 protoc 的插件。前者可以通过该网址：https://github.com/grpc/grpc-web/releases 下载；后者则可以通过：
```shell script
npm install ts-protoc-gen
```

直接下载。

其次，`grpc-web` 需要使用一个 `proxy`，通过反向代理将 `grpc-web` 请求转发给 `grpc-server`。这个代理可以使用 `envoy` 或者 `@improbable-eng/grpc-web` 提供
的 [`grpcwebproxy`](https://github.com/improbable-eng/grpc-web/tree/master/go/grpcwebproxy)