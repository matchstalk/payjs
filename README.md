# PayJS SDK for Go

[![Go Report Card](https://goreportcard.com/badge/github.com/qingwg/payjs)](https://goreportcard.com/report/github.com/qingwg/payjs)


使用Golang开发的PayJS SDK，简单、易用。

[PayJS](https://payjs.cn/ref/DWPXBZ)是微信支付个人接口解决方案，感兴趣的可以去官网看下。

这里是SDK的演示地址：[https://payjs.qingwuguo.com](https://payjs.qingwuguo.com)

## TODO

- 商户资料签名验证失败BUG，为了正常使用，暂取消验证报错
- JSAPI支付签名验证失败BUG，为了正常使用，暂取消验证报错
- 付款码支付测试及演示，没有硬件设备无法完成
- JSAPI支付演示，没有设置JSAPI支付目录无法完成
- 小程序支付演示，没有申请小程序无法完成
- 人脸支付测试及演示，没有硬件设备无法完成
- 订单-撤销测试及演示，没有硬件设备无法完成
- 银行编码查询测试及演示
- 演示程序还有一些细节需要完成

## 基本配置及初始化
下面的是伪代码，请自行理解
```go
payjsConfig := &payjs.Config{
    Key:       "PayJS的通信密钥",
    MchID:     "PayJS的商户号",
    NotifyUrl: "异步通知的路由",
}
Pay = payjs.New(payjsConfig)
```


## 基本API使用

- [扫码支付](#扫码支付)
- [付款码支付](#付款码支付)
- [收银台支付](#收银台支付)
- [JSAPI支付](#JSAPI支付)
- [小程序支付](#小程序支付)
- [人脸支付](#人脸支付)
- [订单](#订单)
	- 查询
	- 关闭
	- 撤销
	- 退款
- [异步通知](#异步通知)
- [用户](#用户)
    - 获取浏览器跳转的url
	- 获取openid
	- ~~获取用户资料~~（即将废弃，SDK相应代码已删除）
- [商户资料](#商户)
- [银行编码查询](#银行编码查询)

## 扫码支付
下面的是伪代码，请自行理解
```go
type Request struct {
	TotalFee     int    `json:"total_fee"`       //Y	金额。单位：分
	Body         string `json:"body"`            //N	订单标题
	Attach       string `json:"attach"`          //N	用户自定义数据，在notify的时候会原样返回
	OutTradeNo   string `json:"out_trade_no"`    //Y	用户端自主生成的订单号
}
type Response struct {
	ReturnCode   int    `json:"return_code"`    //Y	1:请求成功，0:请求失败
	Msg          string `json:"msg"`            //N	return_code为0时返回的错误消息
	ReturnMsg    string `json:"return_msg"`     //Y	返回消息
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
	OutTradeNo   string `json:"out_trade_no"`   //Y	用户生成的订单号原样返回
	TotalFee     int    `json:"total_fee"`      //Y	金额。单位：分
	Qrcode       string `json:"qrcode"`         //Y	二维码图片地址
	CodeUrl      string `json:"code_url"`       //Y	可将该参数生成二维码展示出来进行扫码支付
	Sign         string `json:"sign"`           //Y	数据签名 详见签名算法
}
PayNative := Pay.GetNative()
Response, err := PayNative.Create(Request.TotalFee, Request.Body, Request.OutTradeNo, Request.Attach)
```

官方文档：[扫码支付
](https://help.payjs.cn/api-lie-biao/sao-ma-zhi-fu.html)

## 付款码支付（未测试）
下面的是伪代码，请自行理解
```go
type Request struct {
	TotalFee     int    `json:"total_fee"`       //Y	金额。单位：分
	Body         string `json:"body"`            //N	订单标题
	Attach       string `json:"attach"`          //N	用户自定义数据，在notify的时候会原样返回
	OutTradeNo   string `json:"out_trade_no"`    //Y	用户端自主生成的订单号
	AuthCode     string `json:"auth_code"`       //Y	扫码支付授权码，设备读取用户微信中的条码或者二维码信息(注：用户刷卡条形码规则：18位纯数字，以10、11、12、13、14、15开头)
}
type Response struct {
	ReturnCode   int    `json:"return_code"`    //Y	1:请求成功，0:请求失败
	Msg          string `json:"msg"`            //N	return_code为0时返回的错误消息
	ReturnMsg    string `json:"return_msg"`     //Y	返回消息
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
	OutTradeNo   string `json:"out_trade_no"`   //Y	用户生成的订单号原样返回
	TotalFee     int    `json:"total_fee"`      //Y	金额。单位：分
	Sign         string `json:"sign"`           //Y	数据签名 详见签名算法
}
PayMicropay := Pay.GetMicropay()
Response, err := PayMicropay.Create(Request.TotalFee, Request.Body, Request.OutTradeNo, Request.Attach, Request.AuthCode)
```

官方文档：[付款码支付
](https://help.payjs.cn/api-lie-biao/shua-qia-zhi-fu.html)

## 收银台支付
下面的是伪代码，请自行理解
```go
type Request struct {
	TotalFee     int    `json:"total_fee"`       //Y	金额。单位：分
	Body         string `json:"body"`            //N	订单标题
	Attach       string `json:"attach"`          //N	用户自定义数据，在notify的时候会原样返回
	OutTradeNo   string `json:"out_trade_no"`    //Y	用户端自主生成的订单号
	CallbackUrl  string `json:"callback_url"`    //N	用户支付成功后，前端跳转地址。留空则支付后关闭webview
	Auto         bool   `json:"auto"`            //N	auto=1：无需点击支付按钮，自动发起支付。默认手动点击发起支付
	Hide         bool   `json:"hide"`            //N	hide=1：隐藏收银台背景界面。默认显示背景界面（这里hide为1时，自动忽略auto参数）
}
PayCashier := Pay.GetCashier()
requestUrl, err := PayCashier.GetRequestUrl(Request.TotalFee, Request.Body, Request.OutTradeNo, Request.Attach, Request.CallbackUrl, Request.Auto, Request.Hide)
```

官方文档：[收银台支付
](https://help.payjs.cn/api-lie-biao/shou-yin-tai-zhi-fu.html)

## JSAPI支付（签名验证有bug，目前先取消）
下面的是伪代码，请自行理解
```go
type Request struct {
	TotalFee   int    `json:"total_fee"`    //Y	金额。单位：分
    OutTradeNo string `json:"out_trade_no"` //Y	用户端自主生成的订单号，在用户端要保证唯一性
    Body       string `json:"body"`         //N	订单标题
    Attach     string `json:"attach"`       //N	用户自定义数据，在notify的时候会原样返回
    Openid     string `json:"openid"`       //Y	用户openid
}
type Response struct {
	ReturnCode   int    `json:"return_code"`    //Y	0:失败 1:成功
	ReturnMsg    string `json:"return_msg"`     //Y	失败原因
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 侧订单号
	JsApi        JsApi  `json:"jsapi"`          //N	用于发起支付的支付参数
	Sign         string `json:"sign"`           //Y	数据签名
}
PayJS := Pay.GetJs()
Response, err := PayJS.Create(Request.TotalFee, Request.Body, Request.OutTradeNo, Request.Attach, Request.Openid)
```

官方文档：[JSAPI支付
](https://help.payjs.cn/api-lie-biao/jsapiyuan-sheng-zhi-fu.html)

## 小程序支付
下面的是伪代码，请自行理解

小程序发起支付的解决方案有两种，仅供测试使用

- 方案一：使用小程序消息，结合收银台模式，可以解决小程序支付
- 方案二：使用小程序跳转到 PAYJS 小程序，支付后返回（下面代码是方案二）
```go
type Request struct {
	TotalFee   int    `json:"total_fee"`    //Y	金额。单位：分
    OutTradeNo string `json:"out_trade_no"` //Y	用户端自主生成的订单号，在用户端要保证唯一性
    Body       string `json:"body"`         //N	订单标题
    Attach     string `json:"attach"`       //N	用户自定义数据，在notify的时候会原样返回
    Nonce      string `json:"nonce"`        //Y 随机字符串
}
type Response struct {
	MchID      string `json:"mch_id"`       //Y 商户号
	TotalFee   int    `json:"total_fee"`    //Y 金额。单位：分
	OutTradeNo string `json:"out_trade_no"` //Y 用户端自主生成的订单号
	Body       string `json:"body"`         //N 订单标题
	Attach     string `json:"attach"`       //N 用户自定义数据，在notify的时候会原样返回
	NotifyUrl  string `json:"notify_url"`   //N 异步通知地址
	Nonce      string `json:"nonce"`        //Y 随机字符串
	Sign       string `json:"sign"`         //Y 数据签名 详见签名算法
}
PayMiniApp := Pay.GetMiniApp()
// 获取小程序跳转所需的参数
Response, err := PayMiniApp.GetOrderInfo(Request.TotalFee, Request.Body, Request.OutTradeNo, Request.Attach, Request.Nonce)
```

官方文档：[小程序支付
](https://help.payjs.cn/api-lie-biao/xiao-cheng-xu-zhi-fu.html)

## 人脸支付（未测试）
下面的是伪代码，请自行理解
```go
type Request struct {
	TotalFee   int    `json:"total_fee"`    //Y	金额。单位：分
    OutTradeNo string `json:"out_trade_no"` //Y	用户端自主生成的订单号，在用户端要保证唯一性
    Body       string `json:"body"`         //N	订单标题
    Attach     string `json:"attach"`       //N	用户自定义数据，在notify的时候会原样返回
    Openid     string `json:"openid"`       //Y	OPENID
    FaceCode   string `json:"face_code"`    //Y	人脸支付识别码
}
type Response struct {
	ReturnCode   int    `json:"return_code"`    //Y	1:请求成功，0:请求失败
	Msg          string `json:"msg"`            //N	return_code为0时返回的错误消息
	ReturnMsg    string `json:"return_msg"`     //Y	返回消息
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
	OutTradeNo   string `json:"out_trade_no"`   //Y	用户生成的订单号原样返回
	TotalFee     string `json:"total_fee"`      //Y	金额。单位：分
	Sign         string `json:"sign"`           //Y	数据签名 详见签名算法
}
PayFacepay := Pay.GetFacepay()
Response, err := PayFacepay.Create(Request.TotalFee, Request.Body, Request.OutTradeNo, Request.Attach, Request.Openid, Request.FaceCode)
```

官方文档：[人脸支付
](https://help.payjs.cn/api-lie-biao/ren-lian-zhi-fu.html)

## 订单
下面的是伪代码，请自行理解
```go
// 初始化
PayOrder := Pay.GetOrder()
```

#### 查询
```go
type Request struct {
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
}
type Response struct {
	ReturnCode    int    `json:"return_code"`    //Y	1:请求成功 0:请求失败
	MchID         string `json:"mchid"`          //Y	PAYJS 平台商户号
	OutTradeNo    string `json:"out_trade_no"`   //Y	用户端订单号
	PayJSOrderID  string `json:"payjs_order_id"` //Y	PAYJS 订单号
	TransactionID string `json:"transaction_id"` //N	微信显示订单号
	Status        int    `json:"status"`         //Y	0：未支付，1：支付成功
	Openid        string `json:"openid"`         //N	用户 OPENID
	TotalFee      int    `json:"total_fee"`      //N	订单金额
	PaidTime      string `json:"paid_time"`      //N	订单支付时间
	Attach        string `json:"attach"`         //N	用户自定义数据
	Sign          string `json:"sign"`           //Y	数据签名 详见签名算法
}
Response, err := PayOrder.Check(Request.PayJSOrderID)
```
官方文档：[订单-查询
](https://help.payjs.cn/api-lie-biao/ding-dan-cha-xun.html)

#### 关闭
```go
type Request struct {
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
}
type Response struct {
	ReturnCode   int    `json:"return_code"`    //Y	1:请求成功 0:请求失败
	ReturnMsg    string `json:"return_msg"`     //Y	返回消息
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
	Sign         string `json:"sign"`           //Y	数据签名 详见签名算法
}
Response, err := Response, err := PayOrder.Close(Request.PayJSOrderID)
```
官方文档：[订单-关闭
](https://help.payjs.cn/api-lie-biao/guan-bi-ding-dan.html)

#### 撤销（未测试）
```go
type Request struct {
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
}
type Response struct {
	ReturnCode   int    `json:"return_code"`    //Y	1:请求成功 0:请求失败
	ReturnMsg    string `json:"return_msg"`     //Y	返回消息
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
	Sign         string `json:"sign"`           //Y	数据签名 详见签名算法
}
Response, err := PayOrder.Reverse(Request.PayJSOrderID)
```
官方文档：[订单-撤销
](https://help.payjs.cn/api-lie-biao/che-xiao-ding-dan.html)

#### 退款
```go
type Request struct {
	PayJSOrderID string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
}
type Response struct {
	ReturnCode    int    `json:"return_code"`    //Y	1:请求成功 0:请求失败
	ReturnMsg     string `json:"return_msg"`     //Y	返回消息
	PayJSOrderID  string `json:"payjs_order_id"` //Y	PAYJS 平台订单号
	OutTradeNo    string `json:"out_trade_no"`   //N	用户侧订单号
	TransactionID string `json:"transaction_id"` //N	微信支付订单号
	Sign          string `json:"sign"`           //Y	数据签名 详见签名算法
}
Response, err := PayOrder.Refund(Request.PayJSOrderID)
```
官方文档：[订单-退款
](https://help.payjs.cn/api-lie-biao/tui-kuan.html)

## 异步通知
下面的是伪代码，请自行理解
```go
// Message PayJS支付成功异步通知过来的内容
type Message struct {
	ReturnCode    int    `json:"return_code"`    // 必填	1：支付成功
	TotalFee      int    `json:"total_fee"`      // 必填	金额。单位：分
	OutTradeNo    string `json:"out_trade_no"`   // 必填	用户端自主生成的订单号
	PayJSOrderID  string `json:"payjs_order_id"` // 必填	PAYJS 订单号
	TransactionID string `json:"transaction_id"` // 必填	微信用户手机显示订单号
	TimeEnd       string `json:"time_end"`       // 必填	支付成功时间
	Openid        string `json:"openid"`         // 必填	用户OPENID标示，本参数没有实际意义，旨在方便用户端区分不同用户
	Attach        string `json:"attach"`         // 非必填 用户自定义数据
	MchID         string `json:"mchid"`          // 必填	PAYJS 商户号
	Sign          string `json:"sign"`           // 必填	数据签名 详见签名算法
}

// 传入request和responseWriter
PayNotify := Pay.GetNotify(request, responseWriter)

//设置接收消息的处理方法
PayNotify.SetMessageHandler(func(msg notify.Message) {
    //这里处理支付成功回调，一般是修改数据库订单信息等等
    //msg即为支付成功异步通知过来的内容
})

//处理消息接收以及回复
err := PayNotify.Serve()
if err != nil {
    fmt.Println(err)
    return
}

//发送回复的消息
PayNotify.SendResponseMsg()
```

官方文档：[异步通知
](https://help.payjs.cn/api-lie-biao/jiao-yi-xin-xi-tui-song.html)

## 用户
下面的是伪代码，请自行理解
```go
// 初始化
PayUser := Pay.GetUser()
```

#### 获取浏览器跳转的url
```go
type Request struct {
	CallbackUrl string `json:"callback_url"` //Y	接收 openid 的 url。必须为可直接访问的url，不能带session验证、csrf验证。url 可携带最多1个参数，多个参数会自动忽略
}
url, err := PayUser.GetUserOpenIDUrl(Request.CallbackUrl)
```
官方文档：[用户-获取浏览器跳转的url
](https://help.payjs.cn/api-lie-biao/huo-qu-openid.html)

#### 获取openid
```go
// 在callback_url方法内，传入request
openid, err := PayUser.GetUserOpenID(request)
```
官方文档：[用户-获取openid
](https://help.payjs.cn/api-lie-biao/huo-qu-openid.html)

## 商户资料（签名验证有bug，目前先取消）
下面的是伪代码，请自行理解
```go
type Response struct {
	ReturnCode int    `json:"return_code"` //Y	1:请求成功 0:请求失败
	ReturnMsg  string `json:"return_msg"`  //Y	返回消息
	Doudou     int    `json:"doudou"`      //Y	用户豆豆数
	Name       string `json:"name"`        //Y	商户名称
	Username   string `json:"username"`    //Y	用户姓名
	IDcardNo   string `json:"idcardno"`    //Y	身份证号
	JsApiPath  string `json:"jsapi_path"`  //Y	JSAPI 支付目录
	Phone      string `json:"phone"`       //Y	客服电话
	Sign       string `json:"sign"`        //Y	数据签名 详见签名算法
}
PayMch := Pay.GetMch()
Response, err := PayMch.GetMchInfo()
```

官方文档：[商户资料
](https://help.payjs.cn/api-lie-biao/shang-hu-zi-liao.html)

## 银行编码查询（未测试）
下面的是伪代码，请自行理解
```go
type Request struct {
	Bank string `json:"bank"` //Y	银行简写
}
type Response struct {
	ReturnCode int    `json:"return_code"` //Y	1:请求成功 0:请求失败
	ReturnMsg  string `json:"return_msg"`  //Y	返回消息
	Bank       string `json:"bank"`        //Y	银行名称
	Sign       string `json:"sign"`        //Y	数据签名 详见签名算法
}
PayBank := Pay.GetBank()
Response, err := PayBank.GetBankInfo(Request.Bank)
```

官方文档：[银行编码查询
](https://help.payjs.cn/api-lie-biao/yin-xing-bian-ma-cha-xun.html)

## License

[MIT](LICENSE)
