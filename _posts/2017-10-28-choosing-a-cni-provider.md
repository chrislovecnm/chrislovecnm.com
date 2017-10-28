---
title: Choosing a CNI Provider
description: Choosing which CNI provider to deploy in Kubernetes
header: Choosing a CNI Provider for Kuberentes
date:   2017-10-28 10:46:52 MDT
categories: kuberentes kops
---

Container Network Interface(CNI), is a library, under the umbrella of the
a Cloud Native Computing Foundation project. Kubernetes uses CNI as a interface
between network providers and Kubernetes networking.

The focus of this post is to not tell you which provider to use.  The goal is
to educate and provide data so that you can make a decision.

## Why use CNI

CNI is now always used. The default provider, kubenet, is a simple network plugin
that works with various cloud providers.  kubenet is a very basic network provider
and basic is good, but limiting.  For instance, when running kubenet, in AWS Cloud,
you are limited to 50 EC2 instances.  Route tables are used to configure network
traffice between Kubernetes nodes, and Route tables are limited to 50 entries.
Moreover, a cluster cannot be setup in a Private VPC, since that network topology
uses multiple route tables.  Other more advances features, such as, BGP, egress
control, and mesh networking are only available with other CNI providers.

## CNI in kops

At last count kops supports seven different CNI providers besides kubenet.  Choosing
from seven different network providers is a daunting task.  "Which CNI provider
should I use", is one of the most asked questions that I see on the kops slack
channel.

Here is out current list of providers that can be installed out of the box,
sorted in alphabetical order.

- [Calico](http://docs.projectcalico.org/)
- [Canal (Flannel + Calico)](https://github.com/projectcalico/canal)
- [flannel](https://github.com/coreos/flannel)
- [kopeio-vxlan](https://github.com/kopeio/networking)
- [kube-router](https://github.com/cloudnativelabs/kube-router)
- [romana](https://github.com/romana/romana)
- [weave](https://github.com/weaveworks/weave-kube)

All of the CNI providers use a daemonset installation model, where there product
deploys a a Kubernetes Daemonset.

## Summary of the Providers

### Calico

TODO

### Canal

TODO

### flannel

> Flannel is a simple and easy way to configure a layer 3 network fabric designed for Kubernetes.

@BrandonPhillips provided his views on flannel.

> no external database (uses kubernetes API), simple performant works anywhere VXLAN default, can be layered with Calico policy engine (Canal). Oh, and lots of users.

Flannel along with felix is used in CoreOS's Commercial Kuberentes product Techtonic.

### kopeio-vxlan

@justinsb the founder of kope-vxlan provided this:

> Pioneered the model that everyone is now using, No baggage, with  Minimal CNI dependencies.  Currently there is lower adoption.

### kube-router

TODO

### romana

@chrismarino summarized romana

> Romana uses standard layer 3 networking for pod networks.  Romana supports
Kubernetes Network Policy APIs and never uses an overlay, even when a cluster is
split across network availability zones.  Romana is the only CNI provider that
uses native VPC networking across availability zones for HA clusters, delivering
the highest network performance of  any CNI provider. The current release uses
its own etcd cluster, but the next release will optionally allow the Kubernetes
etcd cluster to be used as a datastore.

### weave

TODO

## By the numbers

kops does not track usage numbers of the different CNI providers, and I hope
never does.  When doing product selection, one of the consideration I use is the
number of stars a project has on GitHub.  The number of contributors talks to
how many people are maintaining the code base, and the number of forks shows
the amount of activity other people have.  FIXME that sounds weird.  Why do forks
matter.

- Calico - stars 240, 57 contributors, 126 forks
- Canal - 430, 17, 58
- flannel - 2463, 79, 593
- kopeio-vxlan - 15, 1, 1
- kube-router - 298, 5, 29
- romana - 148, 9, 20
- weave - stars 4832, 52, 393

## Commerical Support

TODO
