---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "使用 Helmfile 解放你的 Helm Chart"
subtitle: ""
summary: "介绍一款小工具：Helmfile"
authors: ["guoxudong"]
tags: ["helm","kubernetes"]
categories: ["kubernetes"]
date: 2020-12-22T15:59:28+08:00
lastmod: 2020-12-22T15:59:28+08:00
draft: false
type: blog
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65gy1glxx3p2nzsj21jk15oh0o.jpg"
---
## 前言

Helm 作为 Kubernetes 的包管理工具和 CNCF 毕业项目，在业界被广泛使用。但在实际使用场景中的一些需求 helm 并不能很好的满足，需要进行一些修改和适配，如同时部署多个 chart、不同部署环境的区分以及 chart 的版本控制。`Helmfile` 就是一个能够很好解决这些问题的小工具。

## 基础介绍

Helmfile 通过 `helmfile.yaml` 文件帮助用户管理和维护众多 helm chart，其最主要作用是：

- 集成在 CI/CD 系统中，提高部署的可观测性和可重复性，区分环境，免去各种 `--set` 造成的困扰。
- 方便对 helm chart 进行版本控制，如指定版本范围、锁定版本等。
- 定期同步，避免环境中出现不符合预期的配置。

### 安装

helmfile 提供了多种安装方式，除了直接在 [release 页面](https://github.com/roboll/helmfile/releases)下载，还可以通过如下方式安装：

- macOS (使用 homebrew): `brew install helmfile`
- Windows (使用 scoop): `scoop install helmfile`
- Archlinux: `pacman -S helmfile` 
- openSUSE: `zypper in helmfile` 

同时还支持作为容器运行，可以非常方便的集成到 CI/CD 系统中：

```bash
# helm 2
$ docker run --rm --net=host -v "${HOME}/.kube:/root/.kube" -v "${HOME}/.helm:/root/.helm" -v "${PWD}:/wd" --workdir /wd quay.io/roboll/helmfile:v0.135.0 helmfile sync

# helm 3
$ docker run --rm --net=host -v "${HOME}/.kube:/root/.kube" -v "${HOME}/.config/helm:/root/.config/helm" -v "${PWD}:/wd" --workdir /wd quay.io/roboll/helmfile:helm3-v0.135.0 helmfile sync
```

### 其他依赖

除了安装 helmfile 以外，还需要安装 `helm`、`kubectl` 以及 helm 插件 [`helm-diff`](https://github.com/databus23/helm-diff)。

helm-diff 安装方式：

```bash
$ helm plugin install https://github.com/databus23/helm-diff
```

### helmfile.yaml

`helmfile.yaml` 是 helmfile 的核心文件，其用来声明所有的配置。下面会简要介绍一下，详细内容见[官方文档](https://github.com/roboll/helmfile#configuration)。

```yaml
# 声明 repo 配置
repositories:
- name: <repo-name>
  # url: repo url
  # 可以设置基础配置 或 tls 认证
  # certFile: certificate 文件
  # keyFile: key 文件
  # username: 用户名
  # password: 密码

# helm 二进制文件的路径
helmBinary: path/to/helm3

# helm 的一些默认设置，这些配置与 `helm SUBCOMMAND` 相同，可以通过这个配置声明一些，默认的配置
helmDefaults:
  tillerNamespace: tiller-namespace  #dedicated default key for tiller-namespace
  tillerless: false                  #dedicated default key for tillerless
  kubeContext: kube-context          #dedicated default key for kube-context (--kube-context)
  cleanupOnFail: false               #dedicated default key for helm flag --cleanup-on-fail
  # additional and global args passed to helm (default "")
  args:
    - "--set k=v"
  # verify the chart before upgrading (only works with packaged charts not directories) (default false)
  verify: true
  # wait for k8s resources via --wait. (default false)
  wait: true
  # time in seconds to wait for any individual Kubernetes operation (like Jobs for hooks, and waits on pod/pvc/svc/deployment readiness) (default 300)
  timeout: 600
  # performs pods restart for the resource if applicable (default false)
  recreatePods: true
  # forces resource update through delete/recreate if needed (default false)
  force: false
  # when using helm 3.2+, automatically create release namespaces if they do not exist (default true)
  createNamespace: true
  ...

# 为 helmfile 中所有的 release 设置相同的 label，可用于为所有 release 标记相同的版本
commonLabels:
  hello: world

# 设置 release 配置（支持多 release）
releases:
  # 远程 chart 示例（chart 已经上传到 remote 仓库）
  - name: vault                            # name of this release
    namespace: vault                       # target namespace
    createNamespace: true                  # helm 3.2+ automatically create release namespace (default true)
    labels:                                # Arbitrary key value pairs for filtering releases
      foo: bar
    chart: roboll/vault-secret-manager     # the chart being installed to create this release, referenced by `repository/chart` syntax
    version: ~1.24.1                       # the semver of the chart. range constraint is supported
    condition: vault.enabled               # The values lookup key for filtering releases. Corresponds to the boolean value of `vault.enabled`, where `vault` is an arbitrary value
    missingFileHandler: Warn # set to either "Error" or "Warn". "Error" instructs helmfile to fail when unable to find a values or secrets file. When "Warn", it prints the file and continues.
    # Values files used for rendering the chart
    values:
      # Value files passed via --values
      - vault.yaml
      # Inline values, passed via a temporary values file and --values, so that it doesn't suffer from type issues like --set
      - address: https://vault.example.com
      # Go template available in inline values and values files.
      - image:
          # The end result is more or less YAML. So do `quote` to prevent number-like strings from accidentally parsed into numbers!
          # See https://github.com/roboll/helmfile/issues/608
          tag: {{ requiredEnv "IMAGE_TAG" | quote }}
          # Otherwise:
          #   tag: "{{ requiredEnv "IMAGE_TAG" }}"
          #   tag: !!string {{ requiredEnv "IMAGE_TAG" }}
        db:
          username: {{ requiredEnv "DB_USERNAME" }}
          # value taken from environment variable. Quotes are necessary. Will throw an error if the environment variable is not set. $DB_PASSWORD needs to be set in the calling environment ex: export DB_PASSWORD='password1'
          password: {{ requiredEnv "DB_PASSWORD" }}
        proxy:
          # Interpolate environment variable with a fixed string
          domain: {{ requiredEnv "PLATFORM_ID" }}.my-domain.com
          scheme: {{ env "SCHEME" | default "https" }}
    # Use `values` whenever possible!
    # `set` translates to helm's `--set key=val`, that is known to suffer from type issues like https://github.com/roboll/helmfile/issues/608
    set:
    # single value loaded from a local file, translates to --set-file foo.config=path/to/file
    - name: foo.config
      file: path/to/file
    # set a single array value in an array, translates to --set bar[0]={1,2}
    - name: bar[0]
      values:
      - 1
      - 2
    # set a templated value
    - name: namespace
      value: {{ .Namespace }}
    # will attempt to decrypt it using helm-secrets plugin
    
  # 本地 chart 示例（chart 保存在本地）
  - name: grafana                            # name of this release
    namespace: another                       # target namespace
    chart: ../my-charts/grafana              # the chart being installed to create this release, referenced by relative path to local helmfile
    values:
    - "../../my-values/grafana/values.yaml"             # Values file (relative path to manifest)
    - ./values/{{ requiredEnv "PLATFORM_ENV" }}/config.yaml # Values file taken from path with environment variable. $PLATFORM_ENV must be set in the calling environment.
    wait: true

# 可以嵌套其他的 helmfiles，支持从本地和远程拉取 helmfile
helmfiles:
- path: path/to/subhelmfile.yaml
  # label 选择器可以过滤需要覆盖的 release
  selectors:
  - name=prometheus
  # 覆盖 value
  values:
  # 使用文件覆盖
  - additional.values.yaml
  # 覆盖单独的 key
  - key1: val1
- # 远程拉取配置
  path: git::https://github.com/cloudposse/helmfiles.git@releases/kiam.yaml?ref=0.40.0
# 如果指向不存在路径，则打印告警错误
missingFileHandler: Error

# 多环境管理
environments:
  # 当没有设置 `--environment NAME` 时，使用 default 
  default:
    values:
    # 内容可以是文件路径或者 key:value
    - environments/default/values.yaml
    - myChartVer: 1.0.0-dev
  # "production" 环境，当设置了 `helmfile --environment production sync` 时
  production:
    values:
    - environment/production/values.yaml
    - myChartVer: 1.0.0
    # disable vault release processing
    - vault:
        enabled: false
    ## `secrets.yaml` is decrypted by `helm-secrets` and available via `{{ .Environment.Values.KEY }}`
    secrets:
    - environment/production/secrets.yaml
    # 当占不到 `environments.NAME.values` 时，可以设置为 "Error", "Warn", "Info", "Debug"，默认是 "Error"
    missingFileHandler: Error

# 分层管理，可以将所有文件合并，顺序为：environments.yaml < - defaults.yaml < - templates.yaml < - helmfile.yaml
bases:
- environments.yaml
- defaults.yaml
- templates.yaml

# API 功能
apiVersions:
- example/v1
```

### Apply

`helmfile apply` 是 helmfile 中最常用命令，体验与 `kubectl apply` 类似，根据 `helmfile.yaml` 中声明的配置可以一键执行相应的动作，如：添加 repo、安装或更新 release 等。

`helmfile.yaml` 如下：

```yaml
repositories:
- name: stable
  url: https://charts.helm.sh/stable

releases:
- name: prom-norbac-ubuntu
  namespace: prometheus
  chart: stable/prometheus
  set:
  - name: rbac.create
    value: false
```

执行 `helmfile apply` 之后，helmfile 会进行如下操作：

1. 添加 `repositories` 中声明的 repo
2. 运行 `helm diff` 进行对比
3. 根据 `release`中声明的配置，安装或更新 chart

效果如下(由于输出内容过多，这里只节选了部分输出)：

```bash
Adding repo stable https://charts.helm.sh/stable
"stable" has been added to your repositories

Comparing release=prom-norbac-ubuntu, chart=stable/prometheus

...

prometheus, prom-norbac-ubuntu-prometheus-server, ServiceAccount (v1) has been added:
-
+ # Source: prometheus/templates/rbac/server-serviceaccount.yaml
+ apiVersion: v1
+ kind: ServiceAccount
+ metadata:
+   labels:
+     component: "server"
+     app: prometheus
+     release: prom-norbac-ubuntu
+     chart: prometheus-11.12.1
+     heritage: Helm
+   name: prom-norbac-ubuntu-prometheus-server
+   namespace: prometheus
+   annotations:
+     {}

Upgrading release=prom-norbac-ubuntu, chart=stable/prometheus
Release "prom-norbac-ubuntu" does not exist. Installing it now.
NAME: prom-norbac-ubuntu
LAST DEPLOYED: Wed Dec 23 11:23:31 2020
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:

...

Listing releases matching ^prom-norbac-ubuntu$
prom-norbac-ubuntu	prometheus	1       	2020-12-23 11:23:31.779328 +0800 CST	deployed	prometheus-11.12.1	2.20.1


UPDATED RELEASES:
NAME                 CHART               VERSION
prom-norbac-ubuntu   stable/prometheus   11.12.1
```

### 模板化

helmfile 和 helm templete 一样可以使用 [Go templates](https://godoc.org/text/template)，同时还有一个特殊的功能 `requiredEnv`，该函数允许声明模板渲染所需的特定环境变量，如果环境变量未设置或为空，则渲染失败返回错误信息。

### 使用环境变量

可以在 helmfile 中直接使用环境变量，使用方式如下：

```yaml
repositories:
- name: your-private-git-repo-hosted-charts
  url: https://{{ requiredEnv "GITHUB_TOKEN"}}@raw.githubusercontent.com/kmzfs/helm-repo-in-github/master/
releases:
  - name: {{ requiredEnv "NAME" }}-vault
    namespace: {{ requiredEnv "NAME" }}
    chart: roboll/vault-secret-manager
    values:
      - db:
          username: {{ requiredEnv "DB_USERNAME" }}
          password: {{ requiredEnv "DB_PASSWORD" }}
    set:
      - name: proxy.domain
        value: {{ requiredEnv "PLATFORM_ID" }}.my-domain.com
      - name: proxy.scheme
        value: {{ env "SCHEME" | default "https" }}
```

## 进阶实践

helm 还有一些进阶使用方式，如：版本控制、环境区分、hook、交互式操作、集成 kustomize 等。这里简单介绍几种，更多功能请看[官方文档](https://github.com/roboll/helmfile)。

### 版本控制

helmfile 支持 [Semver 2.0](https://semver.org/lang/zh-CN/) 的版本号，可以锁定主版本，防止误升级导致的错误。

```yaml
releases:
  - name: vault                            
    namespace: vault                      
    version: ~1.24.1   # 限制版本 >=1.24.1 && < 1.25.0
```

同时还能通过 `helmfile deps` 命令生成 lock 文件，在 CD 时，除非修改 lock 文件，否无法发布新版本。`helmfile.lock` 内容如下：

```yaml
version: v0.135.0
dependencies:
- name: prometheus
  repository: https://charts.helm.sh/stable
  version: 11.12.1
digest: sha256:a5158f1361f2bbc4e73a80a22dd92b44538bdebeb2419658c36e31aa603b05fd
generated: "2020-12-23T16:26:57.42503+08:00"
```

当需要更新时，再次执行 `helmfile deps` 即可。

### 区分环境

这也是个使用率较高的功能，使用 `environments` 配置·。如果不指定 `--environment NAME` 参数，默认使用 `default` 配置。

这里假设有三个文件，`helmfile.yaml`、`production.yaml` 和 `default.yaml`：

```yaml
# helmfile.yaml
environments:
  default:
    values:
    - default.yaml
  production:
    values:
    - production.yaml 

releases:
- name: myapp-{{ .Values.releaseName }} # 根据环境名，可能是 `dev` 或 `prod`
  values:
  - url: { .Values.domain }} # 根据环境名，可能是 `dev.example.com` 或 `prod.example.com`
{{ if eq .Environment.Name "production" }} # 使用 Go template 的
      - values/production-specified-values.yaml
{{ end }}
```

```yaml
# production.yaml
domain: prod.example.com
releaseName: prod
```

```yaml
# default.yaml
domain: dev.example.com
releaseName: dev
```

在执行 `helmfile` 时，只需使用 `--environment` 指定需要安装的环境：

```bash
$ helmfile --environment production apply
```

### Hook

Helmfile hook 是一个每次发布的扩展点，它由以下部分组成：

- `events`
- `command`
- `args`
- `showlogs`

helmfile 在运行时，会触发各种事件，一旦事件触发，相关的 `hook` 就会被执行，目前支持的如下事件：

- `prepare`
- `presync`
- `preuninstall`
- `postuninstall`
- `postsync`
- `cleanup`

下面这个示例，会打印事件触发时的的上下文信息。

```yaml
environments:
  default:
  prod:

releases:
- name: myapp
  chart: mychart
  # *snip*
  hooks:
  - events: ["prepare", "cleanup"]
    showlogs: true
    command: "echo"
    args: ["{{`{{.Environment.Name}}`}}", "{{`{{.Release.Name}}`}}", "{{`{{.HelmfileCommand}}`}}\
"]
```

执行命令，可以看到 command 执行成功：

```bash
$ helmfile -e prod sync
helmfile.yaml: basePath=.

hook[prepare] logs | prod myapp sync
```

这也是个十分好用的功能，可以为不同的事件配置不同的 hook，这样在 CD 出现问题时，通过 hook 可以第一时间收到通知，并快速定位问题。


## 结语

Helmfile 是一个很不错 Helm 生态工具，很大程度上弥补了 Helm 的不足。提高部署的可观测性和可重复性，提高了效率，最终实现 Release AS Code。