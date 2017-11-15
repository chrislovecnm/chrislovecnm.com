---
title: Choosing a CNI Network Provider for Kubernetes
description: Choosing which CNI Network provider to deploy in Kubernetes
header: Choosing a CNI Network Provider for Kubernetes
date:   2017-11-11 19:22:52 MDT
categories: kubernetes cni
image: /img/cni-logo-card.png
  
---

Container Network Interface (CNI), is a library definition and a set of tools,
under the umbrella of the Cloud Native Computing Foundation project. For more
information visit there GitHub
[project](https://github.com/containernetworking/cni). Kubernetes uses CNI as an
interface between network providers and Kubernetes networking.

![CNI Logo](/img/cni-logo.png){:class="img-responsive"}

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

## Choosing a Provider

> Which CNI provider should I use?

The above question is repeatedly asked on the #kops Kubernetes slack channel.
There are many different providers, which have various features and options.
Deciding which provider to use is not a trivial decision.

I am not going to say in this blog post: "Use this provider."  There is not one
provider that meets everyones needs, and there are many different options. The
focus of this post is not to tell you which provider to use.  The goal of this
blog article is to educate and provide data so that you can make a decision.


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
- [Weave Net](https://github.com/weaveworks/weave-kube)

Any of these CNI providers can be used without kops. All of the CNI providers
use a daemonset installation model, where their product deploys a Kubernetes
Daemonset.  Just use kubectl to install the provider on the master, when the K8s
API server has started. Please refer to each projects specific documentation.

## Summary of the Providers

### Calico

Mike Stowe provided a summary of both Calico and Canal.

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

Brandon Phillips' views on flannel.

Flannel is a simple and easy way to configure a layer3 network fabric designed
for Kubernetes. No external database (uses kubernetes API), simple performant
works anywhere VXLAN default, can be layered with Calico policy engine (Canal).
Oh, and lots of users.

Tectonic, CoreOS's Commercial Kubernetes product, uses a combination of
flannel and Felix from Calico, much like Canal.

### kopeio-networking

Justin Santa Barbara the founder of kopeio provided this:

kopeio-networking provides kubernetes-first networking. It was purpose built
for Kubernetes, making full use of the Kubernetes API, and because of that is
much simpler and more reliable than alternatives that were retrofitted. The
VXLAN approach is the most commonly used mode (as used in weave & flannel), but
it also supports layer 2 (as used in calico), with more experimental support
for GRE (the replacement for IPIP), and for IPSEC (for secure configurations).
It does all of this with a very simple codebase


### kube-router

kube-router is a purpose-built network solution for Kubernetes ground up. It
aims to provide operational simplicity and performance. kube-router delivers a
pod networking solution, a service proxy, and network policy enforcer as
all-in-one solution, with single daemon set. Kuber-router uses Kubernetes native
functionality like annotations, pod CIDR allocation by kube-controller-manger.
So it does not have any dependency on a data store and does not implement any
custom solution for pod CIDR allocation to the nodes. kube-router also uses
standard CNI plug-ins so does require any additional CNI plug-in. kube-router is
built on standard Linux networking toolset and technologies like ipset,
iptables, ipvs, and lvs.

Murali Reddy founder of kube-router.

### romana

Chris Marino summarized romana.

Romana uses standard layer 3 networking for pod networks. Romana supports
Kubernetes Network Policy APIs and does not require an overlay, even when a
cluster is split across network availability zones. Romana support various
network toplogies including flat layer 2 and routed layer 3 networks. Routes
between nodes are installed locally and when necessary, distributed to network
devices using eiter BGP or OSPF. In AWS deployments, Romana installs aggregated
routes into the VPC route table to overcome the 50 node limit. This lets Romana
use native VPC networking across availability zones for HA clusters. The
current release uses its own etcd cluster, but the next version will optionally
allow the Kubernetes etcd cluster to be used as a datastore


### Weave Net

Paul Fremantle's synopsis of Weave Net.

Weave Net supports an overlay network that can span different cloud networking
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

1. GitHub Stars - Likes on GitHub
1. GitHub Forks - Number of copies of the repo
1. GitHub Contributors - Number of people with merged code

The activity level of a project is critical.  These are some of the metrics
that I use to judge the activity level.

### GitHub Stars

GitHub Stars are akin to likes on a Social Media platform.

![Project Stars](/img/cni-github-03.png){:class="img-responsive"}

### GitHub Contributors

The number of contributors talks to the number of people maintaining the code
base and documentation.  Active projects have a high number of contributors. 

![Project Contributors](/img/cni-github-01.png){:class="img-responsive"}

### GitHub Forks

The number of forks is a mix of likes and contributors. Contributors typically
have to fork the repo. Other people will fork the project to build a custom
copy, push code to a feature branch that they own, or for various reasons. 

![Project Forks](/img/cni-github-02.png){:class="img-responsive"}


## Support Matrix

Here is a table of different features of each of the CNI providers mentioned.

<table class="custom-table">
  <thead>
    <tr>
      <th>Provider</th>
      <th>Network<br>Model</th>
      <th>Route <br>Distribution</th>
      <th>Network<br>Policies</th>
      <th>Mesh</th>
      <th>External<br>Datastore</th>
      <th>Encryption</th>
      <th>Ingress/Egress<br>Policies</th>
      <th>Commercial<br>Support</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Calico</td>
      <td>Layer 3</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Etcd <sup><a href="#fn1" id="ref1" style="font-weight:100">1</a></sup></td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>Canal</td>
      <td>Layer 2 vxlan</td>
      <td>N/A</td>
      <td>Yes</td>
      <td>No</td>
      <td>Etcd <sup><a href="#fn1" id="ref1" style="font-weight:100">1</a></sup></td>
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
      <td>None</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kopeio-networking</td>
      <td>Layer 2 vxlan <sup><a href="#fn2" id="ref2" style="font-weight:100">2</a></sup></td>
      <td>N/A</td>
      <td>No</td>
      <td>No</td>
      <td>None</td>
      <td>Yes <sup><a href="#fn3" id="ref3" style="font-weight:100">3</a></sup></td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kube-router</td>
      <td>Layer 3</td>
      <td>BGP</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>romana</td>
      <td>Layer 3</td>
      <td>OSPF</td>
      <td>Yes</td>
      <td>No</td>
      <td>Etcd</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>Weave Net</td>
      <td>Layer 2 vxlan <sup><a href="#fn4" id="ref4" style="font-weight:100">4</a></sup>
      </td>
      <td>N/A</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes <sup><a href="#fn5" id="ref5" style="font-weight:100">5</a></sup></td>
      <td>Yes</td>
    </tr>
  </tbody>
</table>

<sup id="fn1">1. Calico and Canal include a feature to connect directly to
Kubernetes, and not use Etcd.<a href="#ref1"></a></sup><br>
<sup id="fn2">2. kopeio CNI provider has three different networking modes:
vlan, layer2, GRE, and IPSEC.<a href="#ref2" title="Jump back to footnote 2 in
the text."></a></sup><br>
<sup id="fn3">3. kopie-network provides encryptions in IPSEC mode, not the default vxlan mode.<a
href="#ref3" ></a></sup><br>
<sup id="fn4">4. Weave Net can operate in AWS-VPC mode without vxlan, but is limited to 50 nodes in EC2.<a
href="#ref4" ></a></sup><br>
<sup id="fn5">5. Weave Net does not have egress rules out of the box.<a href="#ref5" ></a></sup>


### Table Details

#### Network Model

The Network Model with providers is either encapsulated networking such as VXLAN
or unencapsulated layer 2 networking.  Encapsulating network traffic requires
compute to process, so theoretically is slower.  In my opinion, most use cases
will not be impacted by the overhead.  More about VXLAN on
[wikipedia](https://en.wikipedia.org/wiki/Virtual_Extensible_LAN).

#### Route Distribution

For layer 3 CNI providers, route distribution is necssary. Route distribution
is typically via BGP. Route distribution is a nice to have a feature with CNI,
if you plan to build clusters split across network segments. It is an exterior
gateway protocol designed to exchange routing and reachability information on
the Internet. BGP can assist with pod to pod networking between clusters.

#### Network Policies

A kubernetes.io blog post about network policies in 1.8
[here](http://blog.kubernetes.io/2017/10/enforcing-network-policies-in-kubernetes.html).

> Kubernetes now offers functionality to enforce rules about which pods can
communicate with each other using network policies. This feature is has become
stable Kubernetes 1.7 and is ready to use with supported networking plugins. The
Kubernetes 1.8 release has added better capabilities to this feature.

#### Mesh Networking

This feature allows for "pod to pod" networking between Kubernetes clusters.
This technology is not Kubernetes federation, but it pure networking between
pods.

#### Encyption

Encrypting the network control plane, so all TCP and UDP traffic is encrypted.

#### Ingress / Egress Policies

The network policies are both Kubernetes and Non-Kubernetes routing control. For
instance, many providers will allow an administrator to block a pod
communicating with an EC2 instance meta and data service on 169.254.169.254.

## Summary

If you do not need the advanced features that a CNI provider delivers
use kubenet. It is stable, and fast. Otherwise pick one. If you do need run
more than 50 nodes on AWS or need otheradvanced features  make a decision
quickly, don't spend days deciding, and test with your cluster.  File bugs, and
develop a relationship with your network provider.  At this point in time,
networking is not boring in Kubernetes.  It is getting more boring every day!
Monitor test and monitor more.

## Thanks

Appreciate all of the feedback that I received on the pull request for this
[blog post](https://github.com/chrislovecnm/chrislovecnm.com/pull/1).  Also,
all of the summaries that were provided by the different people that work on
the different projects.  
