---
layout: post # 使用post模版

title:  "golang grpc response wrapper"

date:   2022-11-21

categories: [golang]

---

## 原始代码

```go
xxxResp, err := pkg.GetXXXGrpc().YYYMethod(ctx, &proto.YYYReq{...})
if err != nil {
    // todo log
    return err
}
if xxxResp.GetHeader().GetCode() != 0 {
	// todo log
    return nil
}
// biz
```

### PB 定义

```protobuf
message CheckReportPunishReq {
  // todo 请求参数
}

message CheckReportPunishResp {
  RpcHeader header = 1;
  // todo 返回结果
}

message RpcHeader {
  int32 code = 1;
  string message = 2;
}
```

## 封装代码

```
func GrpcWrapperNoErr[Req, Resp any, RResp interface {
	*Resp
	GetHeader() *proto.RpcHeader
}](ctx context.Context, f func(context.Context, *Req, ...grpc.CallOption) (*Resp, error), req *Req, opts ...grpc.CallOption) *Resp {
	data, err := f(ctx, req, opts...)
	if err != nil {
		// log
		ret := new(Resp)
		wrapErr := reflect.ValueOf(&proto.RpcHeader{Code: 500, Message: "system_busy"})
		reflect.ValueOf(ret).Elem().FieldByName("Header").Set(wrapErr)
		return ret
	}
	pData := RResp(data)
	if pData.GetHeader().GetCode() != 0 {
		// log
		return data
	}
	// debug log
	return data
}
```
### 业务调用

```
uid = utils.GrpcWrapperNoErr(ctx, pkg.GetXXXGrpc().YYY, &proto.YYYReq{
    // xxx
    // yyy
}).GetYYY().GetZZZ()
```

那么就可以直接获取uid了