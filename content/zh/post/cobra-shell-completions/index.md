---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Cobra 命令自动补全指北"
subtitle: ""
summary: "本篇文章就来讲讲如何使用 Cobra 来实现命令自动补全。"
authors: ["guoxuodng"]
tags: ["Go","cobra"]
categories: ["Go"]
date: 2020-08-12T16:48:34+08:00
lastmod: 2020-08-12T16:48:34+08:00
draft: false
type: blog
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65gy1gho5iaurclj20k00b9gsk.jpg"
---
## 前言

用过类 Unix 系统中 Unix shell(Shell/Bash/Zsh) 的同学都应该对 <kbd>TAB</kbd> 键印象深刻，因为它可以帮忙补全或提示后续的命令，用户不用记住完整的命令，只需输入前几个字符，按 <kbd>TAB</kbd> 键，就会提示后续的命令供用户选择，用户体验极佳。目前流行的一些使用 Go 语言开发的 CLI 工具，如 `kubectl` 和 `helm`，他们也都有 `completion` 也就是命令自动补全功能，通过将 `source <(kubectl completion zsh)` 加入 `.zshrc` 文件中，就可以在每次启动 shell 时自动加载自动补全脚本，之后就可以体验到与原生 shell 相同的自动补全功能了。这些 CLI 工具，都是基于 [Cobra](https://github.com/spf13/cobra) 库开发，命令自动补全功能也是该库提供的一个功能，本篇文章就来讲讲如何使用 Cobra 实现命令自动补全的。

## Cobra Shell Completion

Cobra 可以作为一个 Golang 包，用来构建功能强大的命令行程序；同时也可以作为 CLI 工具，用来生成应用程序和命令文件。

由于文本主要介绍 Cobra 的命令自动补全功能，更多内容请查阅[官网](https://github.com/spf13/cobra)。

![Cobra](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gho68w1h0sj20tn09e0td.jpg)

### 基础用法

Cobra 当前的最新版本为 `v1.0.0`，支持生成多种 Shell 的自动补全脚本，目前支持：

- Bash
- Zsh
- Fish
- PowerShell

如上所述，Cobra 不但是一个功能强大的 Golang 包，还是一个 CLI 工具，可以用来生成应用程序和命令文件。使用如下命令，即可生成用于命令自动补全的代码：

```bash
$ cobra add completion
```

或者也可以创建 `cmd/completion.go` 文件，来放置用于生成命令自动补全脚本的代码：

```go
var completionCmd = &cobra.Command{
    Use:                   "completion [bash|zsh|fish|powershell]",
    Short:                 "Generate completion script",
    Long: `To load completions:

Bash:

$ source <(yourprogram completion bash)

# To load completions for each session, execute once:
Linux:
  $ yourprogram completion bash > /etc/bash_completion.d/yourprogram
MacOS:
  $ yourprogram completion bash > /usr/local/etc/bash_completion.d/yourprogram

Zsh:

# If shell completion is not already enabled in your environment you will need
# to enable it.  You can execute the following once:

$ echo "autoload -U compinit; compinit" >> ~/.zshrc

# To load completions for each session, execute once:
$ yourprogram completion zsh > "${fpath[1]}/_yourprogram"

# You will need to start a new shell for this setup to take effect.

Fish:

$ yourprogram completion fish | source

# To load completions for each session, execute once:
$ yourprogram completion fish > ~/.config/fish/completions/yourprogram.fish
`,
    DisableFlagsInUseLine: true,
    ValidArgs:             []string{"bash", "zsh", "fish", "powershell"},
    Args:                  cobra.ExactValidArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      switch args[0] {
      case "bash":
        cmd.Root().GenBashCompletion(os.Stdout)
      case "zsh":
        cmd.Root().GenZshCompletion(os.Stdout)
      case "fish":
        cmd.Root().GenFishCompletion(os.Stdout, true)
      case "powershell":
        cmd.Root().GenPowerShellCompletion(os.Stdout)
      }
    },
}
```

官方推荐将生成内容输出到 `os.Stdout`，只需上面这些简单的命令，即可在你的 CLI 工具中新增 `completion` 子命令，执行该命令即可生成相应 Shell 的命令自动补全脚本，将其插入或保存到相应 Shell 的指定位置即可实现命令自动补全功能。

{{% alert title="注意" color="warning" %}}
如果加载了配置文件，`os.Stdout` 可能会打印多余的信息，这会导致自动补全脚本失效，所以请避免这种情况。
{{% /alert %}}

### 进阶用法

上面的这些只是基本用法，完成的只是命令补全的基本功能，但一些定制化的需求是无法实现的。比如，`kubectl get [tab]` 这里的预期内容是返回所有 k8s 资源名称，但是只靠上面的代码是无法实现的。这里就需要用到自定义补全，通过为每个命令增加不同的参数或方法，可以实现静态和动态补全等功能。

#### 名称补全

名称补全其实也分静态名称和动态名称，静态名称就像 `kubectl completion [tab]` 预期返回的多种 shell 名称，内容为事先在代码中已经定义好的内容；而动态名称，就是像 `helm status [tab]` 预期返回的所有 release 名称，并不是以静态内容体现，而是通过函数动态获取的内容。

##### 静态名称补全

静态名称补全比较简单，只要在想要自动补全的子命令中加入 `ValidArgs` 字段，传入一组包含预期结果的字符串数组即可，代码如下：

```go
validArgs []string = { "pod", "node", "service", "replicationcontroller" }

cmd := &cobra.Command{
    Use:     "get [(-o|--output=)json|yaml|template|...] (RESOURCE [NAME] | RESOURCE/NAME ...)",
    Short:   "Display one or many resources",
    Long:    get_long,
    Example: get_example,
    Run: func(cmd *cobra.Command, args []string) {
      err := RunGet(f, out, cmd, args)
      util.CheckErr(err)
    },
    ValidArgs: validArgs,
}
```

这里是模仿 kubectl 的 `get` 子命令，在执行该命令时效果如下：

```shell
$ kubectl get [tab][tab]
node   pod   replicationcontroller   service
```

如果命令有别名（Aliases）的话，则可以使用 `ArgAliases`，代码如下：

```go
argAliases []string = { "pods", "nodes", "services", "svc", "replicationcontrollers", "rc" }

cmd := &cobra.Command{
    ...
    ValidArgs:  validArgs,
    ArgAliases: argAliases
}
```

{{% alert title="注意" color="warning" %}}
别名不会在按 <kbd>TAB</kbd> 时提示给用户，但如果手动输入，则补全算法会将其视为有效参数，并提供后续的补全。
{{% /alert %}}

```shell
$ kubectl get rc [tab][tab]
backend        frontend       database
```

这里如果不声明 `rc` 为别名，则补全算法将无法补全后续的内容。

##### 动态名称补全

如果需要补全的名称是动态生成的，例如 `helm status [tab]` 这里的 `release` 值，就需要用到 `ValidArgsFunction` 字段，将需要返回的内容以 function 的形式声明在 `cobra.Command` 中，代码如下：

```go
cmd := &cobra.Command{
    Use:   "status RELEASE_NAME",
    Short: "Display the status of the named release",
    Long:  status_long,
    RunE: func(cmd *cobra.Command, args []string) {
      RunGet(args[0])
    },
    ValidArgsFunction: func(cmd *cobra.Command, args []string, toComplete string) ([]string, cobra.ShellCompDirective) {
      if len(args) != 0 {
        return nil, cobra.ShellCompDirectiveNoFileComp
      }
      return getReleasesFromCluster(toComplete), cobra.ShellCompDirectiveNoFileComp
    },
}
```

上面这段代码是 `helm` 的源码，也是 Cobra 的官方示例代码，很好的展示了这个 function 的结构及返回格式，有兴趣的同学可以去看一下 `helm` 的源码，也是很有意思的。`getReleasesFromCluster` 方法是用来获取 Helm release 列表，在执行命令时，效果如下：

```shell
$ helm status [tab][tab]
harbor notary rook thanos
```

`cobra.ShellCompDirective` 可以控制自动补全的特定行为，你可以用或运算符来组合它们，像这样 `cobra.ShellCompDirectiveNoSpace | cobra.ShellCompDirectiveNoFileComp`，下面是它们的介绍（摘自官方文档）：

```go
// Indicates that the shell will perform its default behavior after completions
// have been provided (this implies none of the other directives).
ShellCompDirectiveDefault

// Indicates an error occurred and completions should be ignored.
ShellCompDirectiveError

// Indicates that the shell should not add a space after the completion,
// even if there is a single completion provided.
ShellCompDirectiveNoSpace

// Indicates that the shell should not provide file completion even when
// no completion is provided.
ShellCompDirectiveNoFileComp

// Indicates that the returned completions should be used as file extension filters.
// For example, to complete only files of the form *.json or *.yaml:
//    return []string{"yaml", "json"}, ShellCompDirectiveFilterFileExt
// For flags, using MarkFlagFilename() and MarkPersistentFlagFilename()
// is a shortcut to using this directive explicitly.
//
ShellCompDirectiveFilterFileExt

// Indicates that only directory names should be provided in file completion.
// For example:
//    return nil, ShellCompDirectiveFilterDirs
// For flags, using MarkFlagDirname() is a shortcut to using this directive explicitly.
//
// To request directory names within another directory, the returned completions
// should specify a single directory name within which to search. For example,
// to complete directories within "themes/":
//    return []string{"themes"}, ShellCompDirectiveFilterDirs
//
ShellCompDirectiveFilterDirs
```

{{% alert title="注意" color="warning" %}}
`ValidArgs` 和 `ValidArgsFunction` 同时只能存在一个。在使用 `ValidArgsFunction` 时，Cobra 将在解析了命令行中提供的所有 flag 和参数之后才会调用您的注册函数。
{{% /alert %}}

#### Flag 补全

##### 指定必选 flag

大多时候，名字补全只会提示子命令的补全，但如果一些 flag 是必须的，也可以在用户按 <kbd>TAB</kbd> 键时进行自动补全，代码如下：

```go
cmd.MarkFlagRequired("pod")
cmd.MarkFlagRequired("container")
```

然后在执行命令时，就可以看到：

```shell
$ kubectl exec [tab][tab]
-c            --container=  -p            --pod=  
```

##### 动态 flag

同名称补全类似，Cobra 提供了一个字段来完成该功能，需要使用 `command.RegisterFlagCompletionFunc()` 来注册自动补全的函数，代码如下：

```go
flagName := "output"
cmd.RegisterFlagCompletionFunc(flagName, func(cmd *cobra.Command, args []string, toComplete string) ([]string, cobra.ShellCompDirective) {
    return []string{"json", "table", "yaml"}, cobra.ShellCompDirectiveDefault
})
```

`RegisterFlagCompletionFunc()` 是通过 `command` 与该 flag 的进行关联的，在本示例中可以看到：

```shell
$ helm status --output [tab][tab]
json table yaml
```

使用方式和名称补全相同，这里就不做详细介绍了。

#### Debug

命令自动补全与其他功能不同，调试起来比较麻烦，所以 Cobra 提供了调用隐藏命令，模拟自动补全脚本的方式来帮助调试代码，你可以直接使用以下隐藏命令来模拟触发：

```shell
$ helm __complete status har[ENTER]
harbor
:4
Completion ended with directive: ShellCompDirectiveNoFileComp # This is on stderr
```

{{% alert title="注意" color="warning" %}}
如果需要提示名称而非补全（就是输入命令后直接按 <kbd>TAB</kbd> 键），则必须将空参数传递给 `__complete` 命令：
{{% /alert %}}

```shell
$ helm __complete status ""[ENTER]
harbor
notary
rook
thanos
:4
Completion ended with directive: ShellCompDirectiveNoFileComp # This is on stderr
```

同样可以用来调试 flag 的自动补全：

```shell
$ helm __complete status --output ""[ENTER]
json
table
yaml
:4
Completion ended with directive: ShellCompDirectiveNoFileComp # This is on stderr
```

## 结语

以上内容是作者挑选的一些较为常用的功能，更多的内容详见[官方文档](https://github.com/spf13/cobra)。如果想看示例的话，推荐 [kubectl](https://github.com/kubernetes/kubectl) 和 [helm](https://github.com/helm/helm) 的源码。

当然 Cobra 还不是完美的，比如生成的 Zsh 脚本有些问题，`kubectl` 和 `helm` 都是使用将其生成的 Bash 自动补全脚本转化为 Zsh 的自动补全脚本的方式。但不得不承认，Cobra 是一个非常好用的 CLI 工具构建框架，很多流行的 CLI 工具都是使用它来构建的，这也是为什么使用 GO 语言编写的 CLI 工具如雨后春笋般快速的出现并占据了云原生工具的关键位置。

## 参考

- [Cobra - github.com](https://github.com/spf13/cobra)