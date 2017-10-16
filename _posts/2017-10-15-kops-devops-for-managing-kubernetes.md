---
title: DevOps for Kuberentes Operation
description: kops - Kuberentes Operation is DevOps for Kuberentes clusters.
header: kops DevOps for Kuberetes
date:   2017-10-15 18:50:06 MDT
categories: kubernetes kops
tags:
  - kubernetes
  - kops
  - DevOps
---

[kops](https://github.com/kubernetes/kops) is an Open Source top level
Kubernetes project.  kops is an abbreviation for Kuberentes Operations. From
the README in the project:

> kops helps you create, destroy, upgrade and maintain production-grade, highly
available, Kubernetes clusters from the command line. AWS (Amazon Web Services)
is currently officially supported, with GCE and VMware vSphere in alpha and
other platforms planned.

Founded by the Kuberetes Team in 2016. [Justin Santa
Barbara](https://github.com/justinsb) said that he did not find kops, but it
found us.  [Mike Danese](https://github.com/mikedanese), [Zach
Loafman](https://github.com/zmerlynn), Justin, and many other people of the
Kubernetes team contributed to the intial creation of kops.

## kops is DevOps for Kubernetes

[Wikipedia's](https://en.wikipedia.org/wiki/DevOps) DevOps Definition

> DevOps (a clipped compound of "development" and "operations") is a software
engineering practice that aims at unifying software development (Dev) and
software operation (Ops). The main characteristic of the DevOps movement is to
strongly advocate automation and monitoring at all steps of software
construction, from integration, testing, releasing to deployment and
infrastructure management.

The above definition describes the kops team's vision for developing kops.
Repeatable, tested automation is the core of the philosopy of DevOps.  Through
code the kops' team does the hard things.

## It is like kubectl

Another analogy is think of kops as kubectl, but for clusters.  With kubectl you
can create, update, destroy API components on Kuberentes, from the command line.

For example, to get the pods in the default namespace.

{% highlight bash %}
kubectl get po
{% endhighlight %}

With kops, you can get the clusters that are currently deployed.

{% highlight bash %}
kops get cluster
{% endhighlight %}

## Tutorials

I am not going to cover how to install clusters, in this post, because the kops
project has multiple different tutorials:

- [Amazon Web Services](https://github.com/kubernetes/kops/blob/master/docs/aws.md)
- [Google Compute Engine](https://github.com/kubernetes/kops/blob/master/docs/tutorial/gce.md)

Google Compute Engine is still under development, and with the 1.8.0 release
will be stable.  As of the time of writing this post 1.8.0 is not
[released](https://github.com/kubernetes/kops/releases) yet. Either compile
master or use the 1.8.0 alpha.

## kops Features

- Deploys Highly Available (HA) Kubernetes Masters
- Ability to generate configuration files for Terraform and AWS CloudFormation
- Supports custom Kubernetes add-ons
- Command line autocompletion
- Manifest Based API Configuration
- Upgrade and update managed Kuberentes Clusters
- Validate that Kubernetes is up and running
- Export kubeconfig for a specific installation
- CLI options for edit, create, delete, update, upgrade, rolling-update,
  validate, and more

## kops Concepts

- cluster
- instance group
- create edit update roll
