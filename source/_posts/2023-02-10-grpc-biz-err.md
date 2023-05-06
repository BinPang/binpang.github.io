---
layout: post # 使用post模版

title:  "golang grpc response biz err"

date:   2022-11-21

categories: golang

---

## 背景

我们的服务调用提供的方式如下图

![useful image]({{ site.url }}/assets/2023-02-10/basic.png)

我们也遇到了服务层怎么返回错误到接入层产生了分歧，目前有的3种方案解决

### 通用方案

#### err
直接在err中返回，这里我们需要自定义err，并实现`GRPCStatus`方法
```go
type ERR struct {
    Code uint32
    Msg  string
}
func (p ERR) GRPCStatus() *status.Status {
    return status.New(codes.Code(p.Code), p.Msg)
}
func (p ERR) Error() string {
    return p.Msg
}
```

在返回时使用
```go
func (s *grpcServer) SayHello(ctx context.Context, in *proto.HelloRequest) (*proto.HelloReply, error) {
    return nil, ERR{
        Code: 100_20_01,
        Msg:  "add_reach_max",
    }
}
```

使用这种方式有如下优缺点 

优点：复用grpc标准的错误，简单方便

缺点：耦合了grpc标准的错误，业务上需要区分是grpc的错误，还是业务错误，grpc gateway可能会修改code


#### message
```protobuf
message BatchUserInfoResp {
  RpcHeader header = 1;
  repeated UserInfo infos = 2;
}
```

这里需要引入
```protobuf
message RpcHeader {
  int32 code = 1;
  string message = 2;
}
```



#### metadata+middleware
