---
date: 2016-01-01T00:00:00+00:00
title: How to register your plugin
tags: [ "plugins" ]
labels: [ "tutorial" ]
author: bradrydzewski
---

This tutorial provides an overview for registering your plugin with the [plugin index](http://plugins.drone.io). The plugin index will automatically generate and refresh your documentation from a `manifest.yaml` file in the root of your GitHub repository.

Example `maifest.yaml` file:

```
version: 1

name: Amazon S3
logo: amazon_s3.svg
tags: [ aws, amazon, s3 ]
license: MIT
author: bradyrdzewski
image: plugins/s3

description: >
  The S3 plugin uploads files and build artifacts to your S3 bucket.

settings:
  - name: endpoint:
    desc: custom endpoint URL
  - name: access_key:
    desc: amazon key
    env: AWS_ACCESS_KEY_ID
  - name: secret_key:
    desc: amazon secret key
    env: AWS_SECRET_ACCESS_KEY

example: |
  pipeline:
    s3:
      image: plugins/s3
      bucket: my-bucket-name
      access_key: a50d28f4dd477bc184fbd10b376de753
      secret_key: bc5785d3ece6a9cdefa42eb99b58986f9095ff1c
      source: public/**/*
      target: /target/location

examples:
  - desc: Override the default acl and region:
    code: >
      pipeline:
        s3:
          image: plugins/s3
          bucket: my-bucket-name
      +   acl: public-read
      +   region: us-east-1
          source: public/**/*
          target: /target/location

  - desc: Override the default acl and region:
    code: >
      pipeline:
        s3:
          image: plugins/s3
          bucket: my-bucket-name
          source: public/**/*
          target: /target/location
      +   strip_prefix: public/
```

The `name` attribute provides a user-friendly name for your plugin.

```diff
name: Amazon S3
```

The `logo` attribute is an image in the [repository](https://github.com/drone/drone-plugin-index/tree/master/static/logos) used as the logo in the user interface. If a suitable svg logo does not existing in the repository, please send a pull request.

```diff
logo: amazon_s3.svg
```

The `tags` attribute are a list of labels used to categorize plugins:

```diff
tags: [ aws, amazon, s3 ]
```

The `license` attribute is the spdx license identifier:

```diff
license: MIT
```

The `image` attribute is name of your docker image:

```diff
image: plugins/s3
```


The `description` attribute should provide a one-sentence description of your plugin.

```nohighlight
description: |
  The S3 plugin uploads files and build artifacts to your S3 bucket.
```

The `example` attribute should provide a basic example configuration using the minimum set of required parameters. Avoid including non-essential parameters in this primary example.

```nohighlight
example: |
  pipeline:
    s3:
      image: plugins/s3
      bucket: my-bucket-name
      access_key: a50d28f4dd477bc184fbd10b376de753
      secret_key: bc5785d3ece6a9cdefa42eb99b58986f9095ff1c
      source: public/**/*
      target: /target/location
```

The `examples` block should be used to provide additional examples, where applicable. These example code blocks are rendered in diff format to help illustrate the progression of configuration changes.

```nohighlight
examples:
  - desc: Override the default acl and region:
    code: >
      pipeline:
        s3:
          image: plugins/s3
          bucket: my-bucket-name
          acl: public-read
      +   region: us-east-1
          source: public/**/*
          target: /target/location

  - desc: Override the default acl and region:
    code: >
      pipeline:
        s3:
          image: plugins/s3
          bucket: my-bucket-name
          source: public/**/*
          target: /target/location
      +   strip_prefix: public/
```

The settings block in the manifest should define all plugin yaml attributes:

```nohighlight
settings:
  - name: endpoint:
    desc: custom endpoint URL
  - name: access_key:
    desc: amazon key
  - name: secret_key:
    desc: amazon secret key
```

The settings block should include secret variable names, where applicable:

```diff
settings:
  - name: endpoint:
    desc: custom endpoint URL
  - name: access_key:
    desc: amazon key
+   env: AWS_ACCESS_KEY_ID
  - name: secret_key:
    desc: amazon secret key
+   env: AWS_SECRET_ACCESS_KEY
```
