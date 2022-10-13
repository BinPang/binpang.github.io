---
layout: post # 使用post模版

title:  "golang 业务 error封装"

date:   2022-10-13

categories: golang

---

这是我的第一篇文章

## 问题产生

###
在业务中，api或者rpc返回的数据通常如下格式

```
type Response struct {
	Code int         `json:"code,omitempty"`
	Msg  string      `json:"msg,omitempty"`
	Data interface{} `json:"data,omitempty"`
}
```

而在开发中，一般返回的结果为`(*model.Info, error)`，而不是返回`(*model.Info, code, msg)`

因此一般会包装一个err，例如
```go
type Error struct { 
	code int
	msg  string
}

func (p *mError) Error() string {
	_, msg := GetErrorCodeMsg(p)
	return msg
}
```

由于我们

但是这会出现的一个问题是

```go
func UserService() {
    _, err := json.Marshal()
    if err != nil {
    } // todo
    _, err := dao.GetXX()
    if err != nil {
    }
}

```

这里就会出问题了，第二个`err`会永远成立`err != nil`， 这里为什么永远成立，可以参考一些文章，这里就不叙述了

## 问题解决

### kk

我们规范使用标准的`err`命名为`stdErr`, 业务的`err`命名为`bizErr`，但是出现的问题是历史大量代码，还是容易出错

而

### 第二版

由于第一版还是没有从根本上解决`err`问题，还导致了err越用越乱，到处都是混用

因此我们考虑从根本解决这个问题

导致问题的根本原因是：返回了自定义的err，我们就考虑能不能不返回自定义err呢


```go
package main

type mError struct { // 注意，这里故意是用小写的，防止流到包外去了
    code int
    msg  string
}

func (p *mError) Error() string {
    _, msg := GetErrorCodeMsg(p)
    return msg
}

func GetErrorCodeMsg(err error) (int, string) {
    var me *mError
    if errors.As(err, &me) {
        return me.code, me.msg
    }
    return defaultCode, err.Error()
}
```

