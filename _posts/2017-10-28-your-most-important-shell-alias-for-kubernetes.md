---
title: Your Most Important Shell Alias For Kubernetes
description: The most import shell alias you will ever use with Kubernetes
header: Your most important shell alias for Kubernetes
date:   2017-10-28 07:48:45 MDT
categories: Kubernetes
image: https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo_with_border.png
tags: kubernetes

---

I have been on a mission as of late to use more [shell
aliases](https://www.computerworld.com/article/2598087/linux/how-to-use-aliases-in-linux-shell-commands.html),
and  [Matt Tucker](https://twitter.com/ultimateboy) at the last Kubernetes
meetup, in Boulder, reminded me of huge time savers for kubectl/Kubernetes
users.

Here is the fantastic time saver that needs to be in your shell profile:

{% highlight bash %}
alias k=kubectl
{% endhighlight %}

For those who are not familiar, kubectl is the command line interface
tool for running commands against  Kubernetes clusters.  Most K8s users
use kubectl a lot.

As mentioned in the [kubectl cheat sheet](https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/),
kubectl supports autocomplete in both the bash and zsh shells.

{% highlight bash %}
# setup autocomplete in bash, bash-completion package should be installed first.
source <(kubectl completion bash)
# setup autocomplete in zsh
source <(kubectl completion zsh)
# for help on setting up autocomplete
kubectl completion -h
{% endhighlight %}

For those who want to have more aliases here are 600 for your viewing pleasure:
[https://github.com/ahmetb/kubectl-aliases](https://github.com/ahmetb/kubectl-aliases).

A humorous story, one of the bugs I found in the PetSet documentation was that
the author documented using `k` instead of the `kubectl` command.
