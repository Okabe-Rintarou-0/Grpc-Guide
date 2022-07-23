# NodeJs
这是 NodeJs 使用 grpc 的指南。

## 工具

+ nodeJs
+ npm
+ protoc

## 推荐资料

+ [在 Node.js 中使用 gRPC](https://zhuanlan.zhihu.com/p/163705754)
    
    > 在 node.js 中有两个版本的 grpc，一个是 c++ 版本，一个是纯 js 实现的版本， 对应的 npm 包分别是 grpc 和 @grpc/grpc-js 。两者在接口和功能上基本上没什么差别，显而易见，当然是 c++ 版本的性能更好，但 c++ 版本的在 2021 年就不维护了，另一方面 js 版本的好处则是更方便调试点。

+ [grpc-node github](https://github.com/grpc/grpc-node)

+ [quick start](https://grpc.io/docs/languages/node/quickstart/)

## 使用说明

如果你尝试在 nodeJs 中使用 grpc，那么可以直接参考推荐资料中的教程。推荐使用 `@grpc/grpc-js` 包。只需保证安装了 npm，并且进入了项目的根目录，
使用指令：

```shell script
npm install @grpc/grpc-js 
```

即可。

                                                         