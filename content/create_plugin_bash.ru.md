---
date: 2016-01-01T00:00:00+00:00
title: Creating custom plugins with Bash
tags: [ "bash", "plugins" ]
labels: [ "tutorial" ]
author: AlekSi
---

This provides a brief tutorial for creating a Drone webhook plugin, using simple shell scripting, to make an http requests during the build pipeline. The below example demonstrates how we might configure a webhook plugin in the Yaml file:

```yaml
pipeline:
  webhook:
    image: foo/webhook
    url: http://foo.com
    method: post
    body: |
      hello world
```

Create a simple shell script that invokes curl using the Yaml configuration parameters, which are passed to the script as environment variables in uppercase and prefixed with `PLUGIN_`.

```sh
#!/bin/sh

curl \
  -X ${PLUGIN_METHOD} \
  -d ${PLUGIN_BODY} \
  ${PLUGIN_URL}
```

Allow your users to optionally define sensitive configuration parameters external to the Yaml file, using the Drone secret store. Secrets defined using the secret store use arbitrary names chosen by the plugin author, and are not prefixed with `PLUGIN_`

```diff
#!/bin/sh

+[ -n PLUGIN_USER ] && PLUGIN_USER="${WEBHOOK_USER}"

curl \
  -X "${PLUGIN_METHOD}" \
  -d "${PLUGIN_BODY}" \
+ -u "${PLUGIN_USER}" \
  ${PLUGIN_URL}
```

Create a Dockerfile that adds your shell script to the image, and configures the image to execute your shell script as the main entrypoint.

```dockerfile
FROM alpine
ADD script.sh /bin/
RUN chmod +x /bin/script.sh
RUN apk -Uuv add curl ca-certificates
ENTRYPOINT /bin/script.sh
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
