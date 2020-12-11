# Kubernetes Command Line Tools

[原文](https://blogs.oracle.com/cloudnative/kubernetes-command-line-tools)

## Introduction

如果您经常与Kubernetes集群交互，那么您可能会发现自己被花费在输入重复命令上的时间所困扰。减少这种挫折感的一种方法是使用针对kubectl的CLI工具，即Kubernetes命令行接口。本文将重点介绍几个用于简化kubectl使用并节省时间的工具。

- [shell autocompletion](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion): Autocompletion for Kubectl
- [kubectx & kubens](https://github.com/ahmetb/kubectx): 在Kubernetes上下文和命名空间之间来回切换
- [kube-ps1](https://github.com/jonmosco/kube-ps1): Kubernetes prompt for bash and zsh: context/namespace info to your shell prompt
- [kubectl aliases](https://github.com/ahmetb/kubectl-aliases): 创建别名与kubectl交互

Pre-requisites:默认提示符假设您安装了kubectl命令行实用程序和Kubernetes集群。请参阅这篇[文章](https://kubernetes.io/docs/tasks/tools/install-kubectl/)，了解如何安装kubectl。要使用针对Kubernetes的Oracle容器引擎(OKE)访问Kubernetes集群，请遵循这个[友好的指南](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/oke-full/index.html)。

## Shell Autocompletion: Autocompletion for Kubectl

如果您发现自己花费了太多的时间编写长kubectl命令，那么配置shell自动完成是值得的。这类似于bash、zsh等中的shell完成。这个工具非常有用，因为它可以用来自动完成集群中的所有基本kubectl命令以及部署、服务等。自动完成从当前上下文提取信息，使您不必输入可能很长的部署名称。查看[这篇文章](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)，了解如何为kubectl配置shell自动完成。

使用shell自动完成kubectl只需按tab键，同时写出一个命令。

例如，我们可以键入g，然后按tab键即可自动完成get

```shell
$ kubectl g [TAB]
$ kubectl get
```

这意味着当我们输入s时，你会期望按tab键会自动完成service ，但在这种情况下，以s开头的单词有很多选项。

```shell
$ kubectl get s [TAB]
secrets                                signalfxs.config.istio.io
serviceaccounts                        solarwindses.config.istio.io
servicecontrolreports.config.istio.io  stackdrivers.config.istio.io
servicecontrols.config.istio.io        statefulsets.apps
serviceentries.networking.istio.io     statsds.config.istio.io
servicerolebindings.rbac.istio.io      stdios.config.istio.io
serviceroles.rbac.istio.io             storageclasses.storage.k8s.io
services
```

如果您写出了service，您可以通过tab去显示当前名称空间中的所有可用服务

```shell
$ kubectl get service [TAB]
details      helloworld   kubernetes   productpage  ratings      reviews
```

要查看可用命令的整个列表，输入kubectl，后面跟着两次tab。

```shell
$ kubectl [TAB][TAB]
annotate       certificate    create         explain        plugin         set
api-resources  cluster-info   delete         expose         port-forward   taint
api-versions   completion     describe       get            proxy          top
apply          config         diff           label          replace        uncordon
attach         convert        drain          logs           rollout        version
auth           cordon         edit           options        run            wait
autoscale      cp             exec           patch          scale
```

## kubectx & kubens: switch back and forth between Kubernetes contexts & namespaces

如果您发现自己需要在多个Kubernetes上下文和/或多个名称空间之间来回切换，那么使用工具缩短这个过程可能会有所帮助。使用kubectx，您可以轻松列出、切换、重命名和删除上下文。kubens只是列出并在名称空间之间切换。这两种工具还支持tab完成。

查看[这篇文章](https://github.com/ahmetb/kubectx)，了解如何配置kubectx和kubens

此外，还可以通过添加交互式菜单(如fzf)使kubectx和kubens更加方便。fzf是一个命令行模糊查找器，它为您提供一个上下文或名称空间的交互式菜单供您选择

### Using kubectx

 To list contexts:

```shell
$ kubectx
context-ctdazdbmi3t
docker-for-desktop
```

To switch contexts:

```shell
$ kubectx docker-for-desktop
Switched to context "docker-for-desktop".
```

To return to the previous context:

```shell
$ kubectx -
Switched to context "context-ctdazdbmi3t".
```

To rename a context:

```shell
$ kubectx oracle=context-ctdazdbmi3t
Context "context-ctdazdbmi3t" renamed to "oracle".
```

The same commands apply when kubectx is used with fzf. The only difference is the appearance of an interactive menu.

```shell
$ kubectx

> docker-for-desktop
  context-ctdazdbmi3t
  2/2
```

### Using kubens

To list namespaces:

```shell
$ kubens
default
istio-system
kube-public
kube-system
sample
spinnaker
```

To switch namespaces:

```shell
$ kubens spinnaker
Context "context-ctdazdbmi3t" modified.
Active namespace is "spinnaker".
```

To return to the previous namespace:

```shell
$ kubens - 
Context "context-ctdazdbmi3t" modified.
Active namespace is "default"
```

### Using kubectx with fzf

当kubens与fzf一起使用时，应用相同的命令。唯一的区别是交互菜单的外观。

```shell
$ kubens 

> default
  istio-system
  kube-public
  kube-system
  sample
  spinnaker
  6/6
```

## kube-ps1: Kubernetes prompt for bash and zsh

kube-ps1将当前的Kubernetes上下文和名称空间添加到提示字符串中。这个工具使您不必在每次需要检查上下文或确定当前名称空间时输入kubectl config current-context。这是一个简单的更改，可以使与集群的交互更容易一些。

当然，如果您厌倦了查看上下文和名称空间，您可以简单地使用kubeoff在shell会话中关闭kube-ps1。它可以通过kubeon重启。要使它在shell会话中全局生效，请在命令的末尾添加-g标志

查看[这篇文章](https://github.com/jonmosco/kube-ps1)，了解如何配置kube-ps1

在它被安装之后，你的shell看起来应该像这样:

```shell
(⎈ |[context]:[namespace]) $
(⎈ |context-ctdazdbmi3t:default) $
```

## Kubectl Aliases

bash、zsh等的Shell别名可用于缩短任何和所有Shell命令，而不仅仅是与kubectl相关的命令。在这种情况下，您的环境中不需要安装任何其他东西。将与kubectl相关的别名分离到它们自己的文件中，并在您的shell运行时配置(例如bashrc)中进行源化，这是一个很好的做法。要创建自己的一组kubectl别名，首先在$HOME目录中创建一个文件，例如.kube_alias：

```shell
$ vi .kube_alias
```

首先添加一个简单的别名，比如alias k='kubectl'，然后保存文件。使用以下内容编辑。bashrc或。zshrc文件:

```shell
[ -f ~/.kubectl_alias ] && source ~/.kubectl_alias
```

这将告诉您的shell引用您新创建的别名文件。下次在shell中输入k时，它将被视为输入了kubectl

可以创建别名来简化任意数量的命令。例如，如果您发现自己经常输入`kubectl get pods——all-namespaces`来获得在集群中的每个名称空间中运行的pods列表，那么您可以编写一个别名，比如kgpall来缩短命令。在.kube_alias文件中，看起来如下所示

```shell
 alias kpgall="kubectl get pods --all-namespaces"
```

