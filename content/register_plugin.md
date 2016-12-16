---
date: 2016-01-01T00:00:00+00:00
title: How to register your plugin
tags: [ "plugins" ]
labels: [ "tutorial" ]
author: bradrydzewski
---

The plugin index is a [hugo](https://github.com/spf13/hugo) website that is statically generated. This means registering your plugin simply requires adding your documentation to the [plugin index](http://plugins.drone.io) repository. Fork the repository to get started.

# Markdown File

The plugin documentation should be added to `content/<repository>/index.md`. The repository name should be all lowercase. You can copy / paste and modify existing documentation, or optionally use the hugo command line utility to create your documentation from a template.

```nohighlight
hugo new -f yaml <repo>/index.md
```

This is an example command:

```nohighlight
hugo new -f yaml drone-plugin/drone-docker/index.md
```

Please consider providing translated copies of your documentation if you natively speak one of the below languages. Please do not use Google Translate.

```nohighlight
hugo new -f yaml <repo>/index.fr.md
hugo new -f yaml <repo>/index.es.md
hugo new -f yaml <repo>/index.pt.md
hugo new -f yaml <repo>/index.it.md
hugo new -f yaml <repo>/index.zh-ch.md
hugo new -f yaml <repo>/index.zh-tw.md
hugo new -f yaml <repo>/index.ja.md
hugo new -f yaml <repo>/index.ko.md
hugo new -f yaml <repo>/index.ru.md
hugo new -f yaml <repo>/index.hu.md
```

# Markdown Front Matter

The markdown file includes a yaml frontmatter section that defines plugin meta-data. The following fields are required, unless the description states otherwise.

```yaml
---
date: 2016-01-01T00:00:00+00:00
title: Amazon S3
author: drone-plugins
tags: [ amazon, aws, s3, storage ]
repo: drone-plugins/drone-s3
logo: amazon_s3.svg
image: plugins/s3
---
```

date
: date the plugin was created or updated

title
: name of your plugin

author
: github username of plugin author

repo
: github repository name

image
: docker repository name

tags
: used to categorize your project.

logo
: name of logo to display (optional)


# Documentation Guidelines

The plugin documentation should be succinct and include example configurations. The documentation should first include an example configuration with the minimum set of required fields. For example:

```
pipeline:
  s3:
    image: plugins/s3
    bucket: my-bucket-name
    source: public/**/*
    target: /target/location
```

It should then include examples that add optional, but frequently used fields. We recommend displaying these examples is diff format to help illustrate the progression of configuration changes. For example:

```diff
pipeline:
  s3:
    image: plugins/s3
    bucket: my-bucket-name
+   acl: public-read
+   region: us-east-1
    source: public/**/*
    target: /target/location
```

It should __not__ duplicate information from the core documentation. It should also avoid linking to other documentation. This is because the documentation is frequently changing and we want to minimize the potential for broken links.

For example, do not re-document plugin conditions:

```diff
pipeline:
  s3:
    image: plugins/s3
    bucket: my-bucket-name
    source: public/**/*
    target: /target/location
-   when:
-     branch: master
```

# Documenting Parameters

The markdown documentation should include a section that defines a list of available configuration parameters. This list must use markdown definitions. For example:

```nohighlight
# Parameter Reference

endpoint
: custom endpoint URL (optional, to use a S3 compatible non-Amazon service)

access_key
: amazon key (optional)

secret_key
: amazon secret key (optional)

bucket
: bucket name

region
: bucket region (`us-east-1`, `eu-west-1`, etc)
```

# Documenting Secrets

The markdown documentation should include a section that defines a list of available secret environment variables. This list must use markdown definitions. For example:

```nohighlight
# Secret Reference

AWS_ACCESS_KEY_ID
: amazon key

AWS_SECRET_ACCESS_KEY
: amazon secret key
```

# Including a Logo

It is recommended, but not required, that you include an official logo for your plugin. The logo must be in svg format and placed in the `static/logos` folder.
