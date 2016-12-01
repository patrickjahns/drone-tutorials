---
date: 2016-01-01T00:00:00+00:00
title: Creating custom plugins with Go
tags: [ "golang", "plugins" ]
labels: [ "tutorial" ]
author: appleboy
---

This guide provides a brief tutorial for creating a webhook plugin, using the Go programming language, to trigger http requests during the build pipeline. The below example demonstrates how we might configure a webhook plugin in the Yaml file:

```yaml
pipeline:
  webhook:
    image: foo/webhook
    url: http://foo.com
    method: post
    body: hello world
```

Create a Go program that makes an http request using the Yaml configuration parameters, which are passed to the program as environment variables in uppercase, prefixed with `PLUGIN_`.

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

Allow your users to optionally define sensitive configuration parameters external to the Yaml file, using the Drone secret store. Secrets defined using the secret store use arbitrary names chosen by the plugin author, and are not prefixed with `PLUGIN_`

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

Compile your binary on the host machine for the target platform. Compiling on the host machine and adding the binary to the image is considered a best practice because it reduces the overall image size.

{{% alert warn %}}
It is very important to compile using the correct target platform, otherwise your plugin will fail with a cryptic Go runtime error.
{{% /alert %}}

```nohighlight
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o webhook
```

Create a Dockerfile that adds your compiled binary to the image, and configures the image to run your binary as the main entrypoint.

```dockerfile
FROM alpine
ADD webhook /bin/
RUN apk -Uuv add ca-certificates
ENTRYPOINT /bin/webhook
```

Build and publish your plugin to the Docker registry. Once published your plugin can be shared with the broader Drone community.

```nohighlight
docker build -t foo/webhook .
docker push foo/webhook
```

Execute your plugin locally from the command line to verify it is working:

```nohighlight
docker run --rm \
  -e PLUGIN_METHOD=post \
  -e PLUGIN_URL=http://foo.com \
  -e PLUGIN_BODY="hello world" \
  foo/webhook
```
