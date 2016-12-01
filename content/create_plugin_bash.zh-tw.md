---
date: 2016-01-01T00:00:00+00:00
title: 用 Bash 語言建立 Drone 外掛
tags: [ "bash", "plugins" ]
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
    body: |
      hello world
```

建立簡單的 Shell Script 來執行 curl 指令，並且使用 Yaml 設定來當作環境變數，其中變數名稱必須為大寫且前置字串為 `PLUGIN_` 開頭。

```sh
#!/bin/sh

curl \
  -X ${PLUGIN_METHOD} \
  -d ${PLUGIN_BODY} \
  ${PLUGIN_URL}
```

除了 Yaml 設定外，Drone 也提供加密儲存空間讓使用者自行定義變數，外掛開發者可以透過 Drone Secret 指令來設定私有變數，這些變數名稱不需要使用 `PLUGIN_` 當開頭。

```diff
#!/bin/sh

+[ -n PLUGIN_USER ] && PLUGIN_USER="${WEBHOOK_USER}"

curl \
  -X "${PLUGIN_METHOD}" \
  -d "${PLUGIN_BODY}" \
+ -u "${PLUGIN_USER}" \
  ${PLUGIN_URL}
```

建立 Dockerfile 加入 Shell Script 檔案，並且將執行指令寫在 Entrypoint。

```dockerfile
FROM alpine
ADD script.sh /bin/
RUN chmod +x /bin/script.sh
RUN apk -Uuv add curl ca-certificates
ENTRYPOINT /bin/script.sh
```

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
