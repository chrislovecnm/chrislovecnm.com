---
title: Choosing a CNI Provider
description: Choosing which CNI provider to deploy in Kubernetes
header: Choosing a CNI Provider for Kuberentes
date:   2017-10-28 10:46:52 MDT
categories: kuberentes cni
---

Container Network Interface(CNI), is a library, under the umbrella of the  Cloud
Native Computing Foundation project. Kubernetes uses CNI as an interface between
network providers and Kubernetes networking.

> Which CNI provider should I use?

The above question is repeatedly ask on the #kops Kuberentes slack channel.
There are many different providers, which have various features and options.
Deciding which provider to use is not a trivial decision.

I am not going to say in this blog post: "Use this provider."  There is not one
provider that meets everyone's needs, and there are many different options. The
focus of this post is not to tell you which provider to use.  The goal of this
blog article is to educate and provide data so that you can make a decision.

## Why use CNI

Kubernetes default provider networking provider, kubenet, is a simple network
plugin that works with various cloud providers.  kubenet is a very basic network
provider, and basic is good but does not have very many features. Moreover,
kubenet has many  limitations.  For instance, when running kubenet, in AWS
Cloud, you are limited to 50 EC2 instances.  Route tables are used to configure
network traffic between Kubernetes nodes, and Route tables are limited to 50
entries per VPC. Moreover, a cluster cannot be set up in a Private VPC, since
that network topology uses multiple route tables.  Other more advanced features,
such as BGP, egress control, and mesh networking are only available with
different CNI providers.

## CNI in kops

At last count, kops supports seven different CNI providers besides kubenet.
Choosing from seven different network providers is a daunting task.

Here is our current list of providers that can be installed out of the box,
sorted in alphabetical order.

- [Calico](http://docs.projectcalico.org/)
- [Canal (Flannel + Calico)](https://github.com/projectcalico/canal)
- [flannel](https://github.com/coreos/flannel)
- [kopeio-vxlan](https://github.com/kopeio/networking)
- [kube-router](https://github.com/cloudnativelabs/kube-router)
- [romana](https://github.com/romana/romana)
- [weave](https://github.com/weaveworks/weave-kube)

Any of these CNI providers can be used without kops. All of the CNI providers
use a daemonset installation model, where their product deploys a Kubernetes
Daemonset.  Just use kubectl to install the provider on the master, when the K8s
API server has started. Please refer to each projects specific documentation.

## Summary of the Providers

### Calico

Calico provides simple, scalable networking using a pure L3 approach.  It
enables native, unencapsulated networking in environments that support it,
including AWS AZ's and other environments with L2 adjacency between nodes, or in
deployments where it's possible to peer with the infrastructure using BGP, such
as on-premise. Calico also provides a stateless IP-in-IP mode that can be used
in other environments, if necessary. Beyond scalable networking, Project Calico
also offers policy isolation, allowing you to secure and govern your
microservices/container infrastructure using advanced ingress and egress
policies.   With extensive Kubernetes support, you’re able to manage your
policies, in Kubernetes 1.8+.

### Canal

Canal is a CNI provider that gives you the best of Flannel and Project Calico,
providing simple, easy to use/ out of the box VXLAN networking while also
allowing you take advantage of policy isolation with Calico policies.

This provider is a solution for anyone who wants to get up and running while
taking advantage of familiar technologies that they may already be using.

### flannel

> Flannel is a simple and easy way to configure a layer3 network fabric designed
for Kubernetes.

@BrandonPhillips provided his views on flannel.

> no external database (uses kubernetes API), simple performant works anywhere
VXLAN default, can be layered with Calico policy engine (Canal). Oh, and lots of
users.

Techtonics,  CoreOS's Commercial Kuberentes product, uses a combination of
flannel and Felix from Calico, much like Canal.

### kopeio-vxlan

@justinsb the founder of kope-vxlan provided this:

> Pioneered the model that everyone is now using, No baggage, with  Minimal CNI
dependencies.  Currently, there is lower adoption.

### kube-router

Kuber-router is a purpose-built network solution for Kuberentes ground up. It
aims to provide operational simplicity and performance. Kube-router delivers a
pod networking solution, a service proxy, and network policy enforcer as
all-in-one solution, with single daemon set. Kuber-router uses Kubernetes native
functionality like annotations, pod CIDR allocation by kube-controller-manger.
So it does not have any dependency on a data store and does not implement any
custom solution for pod CIDR allocation to the nodes. Kube-router also uses
standard CNI plug-ins so does require any additional CNI plug-in. Kube-router is
built on standard Linux networking toolset and technologies like ipset,
iptables, ipvs, and lvs.

### romana

@chrismarino summarized romana

> Romana uses standard layer 3 networking for pod networks.

Romana supports Kubernetes Network Policy APIs and never uses an overlay, even
when a cluster is split across network availability zones.  Romana is the only
CNI provider that uses native VPC networking across availability zones for HA
clusters, delivering a high-performance CNI solution. The current release uses
its Etcd cluster, but the next version will optionally allow the Kubernetes Etcd
cluster to be used as a datastore.

### weave

Paul Fremantle summary of weave

> Weave supports an overlay network that can span different cloud networking
configurations, simplifying running legacy workloads on Kubernetes. For example,
Weave supports multicast, even when the underlying network doesn't. Weave can
configure the underlying VPC networking and bypass the overlay when running on
AWS.  This provider forms a mesh network of hosts that are partitionable and
eventually consistent, meaning that the setup is almost zero-config, and it
doesn't need to rely on an Etcd. Weave supports encryption and Kubernetes
network policy ensuring that there is security at the network level.

## By the Numbers

kops does not track usage numbers of the different CNI providers, and I hope
never does.  When making a product selection, of software hosted on GitHub, I
look at three different numbers.

1. GitHub Stars - Likes on Github
1. GitHub Forks - Number of copies of the repo
1. GitHub Contributors - Number of people with merged code

Github Stars are akin to likes on a Social Media platform.  The number of
contributors talks to the number of people maintaining the code base and
documentation.  Active projects have a high number of contributors. The number
of forks is a mix of likes and contributors. Contributors typically have to fork
the repo. Other people will fork the project to build a custom copy, push code
to a feature branch that they own, or for various reasons. The activity level of
a project is critical.  These are some of the metrics that I use to judge the
activity level.


### GitHub Stars

![Project Stars](/img/cni-github-03.png){:class="img-responsive"}

### GitHub Forks

![Project Stars](/img/cni-github-02.png){:class="img-responsive"}

### GitHub Contributors

![Project Stars](/img/cni-github-01.png){:class="img-responsive"}

## Support Matix

<table class="custom-table">
  <thead>
    <tr>
      <th>Provider</th>
      <th>Network <br>Model</th>
      <th>BGP</th>
      <th>Network <br>Policies</th>
      <th>Mesh</th>
      <th>External <br>Database</th>
      <th>Encyption</th>
      <th>Ingress / Egress<br> Policies</th>
      <th>Commercial <br>Support</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Calico</td>
      <td>layer3</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes*</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>Canal</td>
      <td>vxlan</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes*</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
    </tr>
    <tr>
      <td>flannel</td>
      <td>vxlan</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kopeio-vxlan</td>
      <td>vxlan*</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kube-router</td>
      <td>layer3</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>romana</td>
      <td>layer3</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>weave</td>
      <td>vxlan*</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
  </tbody>
</table>
