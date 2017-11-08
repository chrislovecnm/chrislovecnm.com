---
title: Choosing a CNI Provider
description: Choosing which CNI provider to deploy in Kubernetes
header: Choosing a CNI Provider for Kuberentes
date:   2017-10-28 10:46:52 MDT
categories: kuberentes cni
---

Container Network Interface(CNI), is a library, under the umbrella of the
a Cloud Native Computing Foundation project. Kubernetes uses CNI as a interface
between network providers and Kubernetes networking.

> Which CNI provider should I use?

The above question is repeatedly ask on the #kops Kuberentes slack channel.
There  are a number of different providers, which have various different
features and options. Deciding which provider to use is not trivial.

I am not going to say in this blog post: "Use this provider."  There is not one
provider that meets everyones needs, and there are many different options.

The focus of this post is to not tell you which provider to use.  The goal of
this blog article is to educate and provide data so that you can make a
decision.

## Why use CNI

Kubernetes default provider networking provider, kubenet, is a simple network
plugin that works with various cloud providers.  kubenet is a very basic network
provider and basic is good, but does not have very many features. Moreover,
kubernete has many limitations.  For instance, when running kubenet, in AWS
Cloud, you are limited to 50 EC2 instances.  Route tables are used to configure
network traffice between Kubernetes nodes, and Route tables are limited to 50
entries per VPC. Moreover, a cluster cannot be setup in a Private VPC, since
that network topology uses multiple route tables.  Other more advances features,
such as, BGP, egress control, and mesh networking are only available with other
CNI providers.

## CNI in kops

At last count kops supports seven different CNI providers besides kubenet.
Choosing from seven different network providers is a daunting task.

Here is out current list of providers that can be installed out of the box,
sorted in alphabetical order.

- [Calico](http://docs.projectcalico.org/)
- [Canal (Flannel + Calico)](https://github.com/projectcalico/canal)
- [flannel](https://github.com/coreos/flannel)
- [kopeio-vxlan](https://github.com/kopeio/networking)
- [kube-router](https://github.com/cloudnativelabs/kube-router)
- [romana](https://github.com/romana/romana)
- [weave](https://github.com/weaveworks/weave-kube)

Any of these CNI providers can be used without kops. All of the CNI providers
use a daemonset installation model, where their product deploys a a Kubernetes
Daemonset.  Just use kubectl to install the provider on the master, when the K8s
API server has started. Please refer to each projects specific documentation.

## Summary of the Providers

### Calico

Project Calico is a widely used and trusted network and security CNI plugins.
Calico provides simple, scalable networking using a pure L3 approach. It enables
native, unencapsulated networking in environments that support it, including AWS
AZ's and other environments with L2 adjacency between nodes, or in environments
where it's possible to peer with the infrastructure using BGP, such as
on-premises). Calico also provides a stateless IP-in-IP mode that can be used in
other environments, if necessary. Beyond scalable networking, Project Calico
also offers policy isolation, allowing you to secure and govern your
microservices/ container infrastructure using advanced ingress and egress
policies.   With extensive Kubernetes support, you’re able to easily manage your
policies (in Kubernetes 1.8+) using the Kubernetes API.

### Canal

Canal is a CNI provider that gives you the best of Flannel and Project
Calico, providing simple, easy to use/ out of the box VXLAN networking while
also allowing you take advantage of policy isolation with Calico policies.

This is a solution for anyone who wants to get up and running quickly
while taking advantage of familiar technologies that they may already be using.


### flannel

> Flannel is a simple and easy way to configure a layer 3 network fabric
designed for Kubernetes.

@BrandonPhillips provided his views on flannel.

> no external database (uses kubernetes API), simple performant works anywhere
VXLAN default, can be layered with Calico policy engine (Canal). Oh, and lots of
users.

Flannel along with felix is used in CoreOS's Commercial Kuberentes product
Techtonic.

### kopeio-vxlan

@justinsb the founder of kope-vxlan provided this:

> Pioneered the model that everyone is now using, No baggage, with  Minimal CNI
dependencies.  Currently there is lower adoption.

### kube-router

Kuber-router is a purpose built network solution for Kuberentes ground up. It
aims to provide opertational simplicity and performance. Kube-router provides a
pod networking solution, a service proxy and network policy enforcer as
all-in-one solution, with single daemon set. Kuber-router uses Kubernetes native
functionality like annotations, pod CIDR allocation by kube-controll-manger. So
it does not have any dependency on external data store and does not implement
any custom solution for pod CIDR allocation to the nodes. Kube-router also uses
standard CNI plug-ins so does required any additional CNI plug-in. Kube-router
is built on standard linux networking toolset and technologies like ipset,
iptables, ipvs/lvs etc

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

Paul Fremantle summary of weave

> Weave supports an overlay network that can span availability zones and other
underlying networks, simplifying running legacy workloads on K8s. For example,
Weave supports multicast, even when the underlying network doesn't. Weave can
configure the underlying VPC networking and bypass the overlay when running on
AWS. Weave forms a mesh network of hosts that is partitionable and eventually
consistent, meaning that the setup is almost zero-config, and it doesn't need to
rely on an etcd. Weave supports encryption and Kubernetes network policy
ensuring that there is security at the network level.  Estimate that there are
>100k Kuberetes users with strong growth from Kuberentes deployments over the
past few months.

## By the numbers

kops does not track usage numbers of the different CNI providers, and I hope
never does.  When doing product selection, one of the consideration I use is the
number of stars a project has on GitHub.  The number of contributors talks to
how many people are maintaining the code base, and the number of forks shows
the amount of activity other people have.  FIXME that sounds weird.  Why do forks
matter.

![Project Stars](/img/cni-github-01.png){:class="img-responsive"}
![Project Stars](/img/cni-github-02.png){:class="img-responsive"}
![Project Stars](/img/cni-github-03.png){:class="img-responsive"}
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
