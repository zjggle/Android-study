### **协议消息头**
```
{
    protocol: 'vetp-issuer',
	version:  '0.1.0',
	type:     'ISSUE',
	createTime: '1411439205591',
	signature:'0123456789abcdef0123456789ab'
}
```
Attributes：

| Name          | Type    | Example   | Description  |
|:------------- |:------: | :-------  | :----------- |
| `protocol`    | string  |vetp-issuer| 协议名称     |
| `version`     | string  | 0.1.0     | 协议版本     |
| `type`        | string  | ISSUE/QUERY| 发送请求的消息类型 [参见](#request_type)     |
| `createTime`        | date-time| "1411439205591"|消息创建时间|
| `signature`   | string  | "0123456789abcdef0123456789ab"|签名，通过GPG对上面数据使用私钥的签名字符串|


返回的消息格式：
```
{
	type:     'RESPONSE', 
	response:  {...}
}
```
Attributes：

| Name          | Type    | Example   | Description  |
|:------------- |:------: | :-------  | :----------- |
| `type`        | string  | RESPONSE  | 返回消息类型 [参见](#response_type)     |
| `response`     | object  | {....}|消息负载，这里表示issuer与validator-network通讯的消息内容|

返回失败的消息格式：
```
{
	type:     'RESPONSE_ERROR', 
	error:  {
        code:    '1000'
	    message: '签名验证失败',
        request: {...}
	}
}
```
Attributes：

| Name          | Type    | Example   | Description  |
|:------------- |:------: | :-------  | :----------- |
| `type`        | string  | RESPONSE_ERROR  | 返回消息类型 [参见]     |
| `code`   | string | 1000      | validator返回处理当前消息的错误编码，[错误编码参见](#error_code)|
| `message`| string | 签名验证失败   | 消息错误内容 |
| `request`     | object  | {....}    |issuer发送请求的消息内容|



---

### **issue接口数据格式**
```
{
    protocol: 'vetp-issuer',
	version:  '0.1.0',
	type:     'ISSUE',
	createTime:    '1411439205591',
	transactionId: '01234567-89ab-cdef-0123-456789abcdef',
	transaction: {
		issuer:   'xxxxxx',
		amount:   '1000',
		currency: 'USD',
		receiver: 'bbbbbb',
	},
	signature:'0123456789abcdef0123456789ab'
}
```
消息体参数说明：

| Name           | Type    | Example   | Description  |
|:-------------  |:------: | :-------  | :----------- |
| `issuer`       | string  | xxxxxx    | 货币发行人   |
| `amount`       |decimal(10,5) | 1000 | 发行的数量，可以是整数或者小数|
| `currency`     | string  | USD       | 发行的货币类型：USD/CNY/自定义货币类型，当前issuer发行的货币类型|
| `receiver`     | object  | bbbbbb    | 当前issuer发行货币的接收人|
| `transactionId`| uuid v4 | '01234567-89ab-cdef-0123-456789abcdef'| 消息去重, UUID V4的方式生成|

validator-network数据返回的格式：
```
{
	type:     'ISSUE_RESPONSE', 
	response: {
		transactionId: '01234567-89ab-cdef-0123-456789abcdef',
		status: 'commit'//返回当前请求消息状态码，[详情参见]常量
	}
}
```

Attributes：

| Name           | Type    | Example   | Description  |
|:-------------  |:------: | :-------  | :----------- |
| `status`       | enum | 'pending'| 消息处理状态initial/pending/commit/empty|


---

### **数据查询接口格式**
```
{
    protocol: 'vetp-issuer',
	version:  '0.1.0',
	type:     'QUERY_BY_ID',
	query:    '01234567-89ab-cdef-0123-456789abcdef',
	signature:'0123456789abcdef0123456789ab',
}
```
Attributes：

| Name           | Type    | Example   | Description  |
|:-------------  |:------: | :-------  | :----------- |
| `query`| uuid v4 | '01234567-89ab-cdef-0123-456789abcdef'| 查询的消息UUID|

返回数据格式：
```
{
    type:     'QUERY_BY_ID_RESPONSE',
    response: {
        type: 'ISSUE',//消息类型，如ISSUE/EXCHANGE...
        transactionId: '01234567-89ab-cdef-0123-456789abcdef',
	    [....]//todo 具体数据格式，接下来会完善
    }
}
```
---

### **消息状态查询接口数据格式**
```
{
    protocol: 'vetp-issuer',
	version:  '0.1.0',
    type:     'QUERY_STATE_BY_ID',
	query:    '01234567-89ab-cdef-0123-456789abcdef',
    signature:'0123456789abcdef0123456789ab'
}
```
返回数据格式:
```
{
    protocol: 'vetp-issuer',
    version:  '0.1.0',
    type:     'QUERY_STATE_RESPONSE',
    payload: {
        transactionId: '01234567-89ab-cdef-0123-456789abcdef',
        type: '',//消息类型，如ISSUE/EXCHANGE...
	    status : 'pending'//状态[参考常量]
    }
}
```

Attributes：

| Name           | Type    | Example   | Description  |
|:-------------  |:------: | :-------  | :----------- |
| `status`       | enum | 'pending'| 消息处理状态initial/pending/commit/empty|


---

<A NAME="request_type">**请求的消息类型**</A>

| Name           | Description  |
|:-------------  |:-------------|
| `ISSUE`        | issuer发行货币|
| `QUERY_BY_ID`        | 查询消息 |
| `QUERY_STATE_BY_ID`  | 查询消息状态 |
| `EXCHANGE`     | exchange消息 |


---

<A NAME="response_type">**validator返回的消息类型**</A>

| Name           | Description  |
|:-------------  |:-------------|
| `ISSUE_RESPONSE` | issuer发行货币|
| `QUERY_BY_ID_RESPONSE` | 查询消息 |
| `QUERY_STATE_BY_IDRESPONSE` | 查询消息状态 |
| `ERROR`   | 返回错误 |


<A NAME="error_code">**消息字典**</A>

| 字典码         | 说明         |
|:-------------  |:-------------|
| `0000`         | 成功|
| `1000`         | 系统异常|
| `1100`         | 数字签名不合法|
| `1101`         | 数据格式不合法 |
| `1102`         | 数据类型不合法 |
| `1200`         | issuer不存在 |
| `1201`         | issuer账户余额不足|
| `1202`         | receiver不存在|
