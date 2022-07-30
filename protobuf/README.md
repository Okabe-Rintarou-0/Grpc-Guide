# Protobuf

![protobuf](protobuf.webp)

## 工具

+ protoc

    下载地址：https://github.com/protocolbuffers/protobuf/releases

## 推荐资料
+ [protobuf github](https://github.com/protocolbuffers/protobuf)
+ [protobuf 官方文档](https://developers.google.cn/protocol-buffers/docs/overview)
+ [Scalar Value Types](https://developers.google.cn/protocol-buffers/docs/proto3#scalar)
+ [grpc github](https://github.com/grpc)
+ [grpc 官网](https://www.grpc.io/)
+ [grpc 官网中文版](http://doc.oschina.net/grpc)

## 使用说明

### 类型定义非常重要

如果你有看过 protobuf 官方文档中对于支持的类型的表格（见推荐资料中的`Scalar Value Types`），你会发现 `string` 和 `bytes` 类型对应的 `c++` 中
的类型均为 `string`。但是两者有很大的不同，如果使用不当可能导致信息丢失或错乱。前者的每个字符都是 `utf-8` 编码的，
必须严格遵守；而对于后者，如果你需要传递的更像是一个字节流，而非 `utf-8` 编码的字符串，那么就必须使用 `bytes`。
