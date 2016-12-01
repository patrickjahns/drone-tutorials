---
date: 2016-01-01T00:00:00+00:00
title: 用 Go 語言建立 Drone 外掛
tags: [ "golang", "plugins" ]
labels: [ "tutorial" ]
author: appleboy
---

本文會教大家如何使用簡單的 Shell Script 建立 Drone webhook 外掛，並在 Drone 編譯過程中建立 http 連線，底下範例是在 Yaml 檔案內設定 webhook 外掛:

```yaml
pipeline:
  webhook:
    image: foo/webhook
    url: http://foo.com
    method: post
    body: hello world
```

建立 Go 程式來執行 http 請求連線，並透過 Yaml 設定來傳遞環境變數，其中變數名稱必須為大寫且前置字串為 `PLUGIN_` 開頭。

```Go
package main

import (
  "net/http"
  "os"
)

func main() {
  body := strings.NewReader(
    os.GetEnv("PLUGIN_BODY"),
  )

  req, err := http.NewRequest(
    os.GetEnv("PLUGIN_METHOD"),
    os.GetEnv("PLUGIN_URL"),
    body,
  )
  if err != nil {
    os.Exit(1)
  }
}
```

除了 Yaml 設定外，Drone 也提供加密儲存空間讓使用者自行定義變數，外掛開發者可以透過 Drone Secret 指令來設定私有變數，這些變數名稱不需要使用 `PLUGIN_` 當開頭。

```diff
package main

import (
  "net/http"
+ "net/url"
  "os"
)

func main() {
  body := strings.NewReader(
    os.GetEnv("PLUGIN_BODY"),
  )

  req, err := http.NewRequest(
    os.GetEnv("PLUGIN_METHOD"),
    os.GetEnv("PLUGIN_URL"),
    body,
  )

+ req.URL.User = url.UserPassword(
+   os.GetEnv("WEBHOOK_USERERNAME"),
+   os.GetEnv("WEBHOOK_PASSWORD"),
+ }

  if err != nil {
    os.Exit(1)
  }
}
```

針對目標平台環境中編譯執行擋。在主機上編譯，並且將執行擋加入 Image，這是最佳實踐，因為它將減少整個映像檔大小。

{{% alert warn %}}
使用正確的 target platform 去編譯是非常重要的，否則您的外掛會失敗在 Go 運行中的隱藏錯誤。
{{% /alert %}}

```nohighlight
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o webhook
```

Create a Dockerfile that adds your compiled binary to the image, and configures the image to run your binary as the main entrypoint.

建立 Dockerfile 加入您編譯過的 binary，並且設定 Entrypoint 讓映像檔去執行您的 binary。 

```dockerfile
FROM alpine
ADD webhook /bin/
RUN apk -Uuv add ca-certificates
ENTRYPOINT /bin/webhook
```

Build and publish your plugin to the Docker registry. Once published your plugin can be shared with the broader Drone community.

編譯並且上傳映像檔到 Docker registry，一旦完成後，就可以分享到 Drone 社群內。

```nohighlight
docker build -t foo/webhook .
docker push foo/webhook
```

在本機端執行您的外掛來確保映像檔是可以正常運作：

```nohighlight
docker run --rm \
  -e PLUGIN_METHOD=post \
  -e PLUGIN_URL=http://foo.com \
  -e PLUGIN_BODY="hello world" \
  foo/webhook
```
