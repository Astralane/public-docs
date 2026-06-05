---
icon: fire
---

# 发送交易 V2

**主要改进**

* 无需触发 CORS 预检请求
* 使用纯文本请求体，替代 JSON
* 显著减小请求体体积，降低带宽与网络开销

> #### 关键说明
>
> * 请求头仅需保留一项：`Content-Type: text/plain`
> * 交易数据需使用 Base64 编码
> * 必须在 URI 参数中携带 `api-key` 与 `method`
> * 当前仅支持 `sendTransaction` 方法
> * 请求路径：`/iris2`

#### URI 参数

<table><thead><tr><th width="171">参数名</th><th width="120">类型</th><th>Description</th></tr></thead><tbody><tr><td><code>api-key</code></td><td>String</td><td><strong>必填</strong>，用于鉴权</td></tr><tr><td><code>method</code></td><td>String</td><td><strong>必填</strong>，用于指定调用方法</td></tr><tr><td><code>mev-protect</code></td><td>Boolean</td><td><strong>选填</strong>，是否开启 MEV 保护，默认 <code>false</code></td></tr></tbody></table>

#### 支持的方法

<table><thead><tr><th width="175">方法名</th><th width="136">HTTP 动词</th><th>说明</th></tr></thead><tbody><tr><td><code>sendTransaction</code></td><td><code>POST</code></td><td>发送 Base64 编码后的交易</td></tr><tr><td><code>getHealth</code></td><td><code>POST</code></td><td>保活探测 / 健康检查</td></tr></tbody></table>

#### 请求示例

```shellscript
curl --location 'https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction' \
--header 'Content-Type: text/plain' \
--data 'base64_encoded_txn'
```

Rust :

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = reqwest::Client::builder()
        .build()?;

    let mut headers = reqwest::header::HeaderMap::new();
    headers.insert("Content-Type", "text/plain".parse()?);

    let data = "base64_encoded_txn";

    let request = client.request(reqwest::Method::POST, "https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction")
        .headers(headers)
        .body(data);

    let response = request.send().await?;
    let body = response.text().await?;

    println!("{}", body);

    Ok(())
}

```

NodeJS

```javascript
const axios = require('axios');
let data = 'base64_encoded_txn';

let config = {
  method: 'post',
  maxBodyLength: Infinity,
  url: 'https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction',
  headers: { 
    'Content-Type': 'text/plain'
  },
  data : data
};

axios.request(config)
.then((response) => {
  console.log(JSON.stringify(response.data));
})
.catch((error) => {
  console.log(error);
});

```

Go Lang

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io"
)

func main() {

  url := "https://lim.gateway.astralane.io/iris2?api-key=APIKEY&method=sendTransaction"
  method := "POST"

  payload := strings.NewReader(`base64_encoded_txn`)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Add("Content-Type", "text/plain")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := io.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

#### 返回结果说明

| HTTP 状态码 | 含义           |
| -------- | ------------ |
| 200      | 请求成功，交易已正常处理 |
| 400      | 请求参数或格式存在问题  |

