---
title: GKE On-Prem
description: GKE On-Prem
header: GKE On-Prem
date:   2018-08-18 12:37:50 MDT
categories: gke on-prem
tags: gke on-prem google next googlenext2018 kubernetes
image: /img/k8s/logo-twitter.png
---

## GoogleNEXT2018 Shocker

The announcement of GKE On-Prem was a big surprise to me. At Google Next 2018 in San Fransico, Google announced the alpha program for GKE On-Prem.  This product is a game changer and puts Google in direct competition with products like OpenShift, Techtonic, and Canonical's distro of Kubernetes.  Google's clients asked for an On-Prem solution where they could run the same distribution of GKE in their data center that Google runs for GKE. That simple point, "Our clients asked for it," speaks volumes to Google Clouds
customer focused nature.

The release of GKE On-Prem was a well-guarded secret inside the walls of Google.
I was involved in a [program]({% post_url 2018-07-25-kubernetes-engine-demos %}) that supported running a hybrid cloud solution which fits that model of running GKE On-Prem within your datacenter.

![K8s Logo](/img/k8s/logo-twitter.png){:class="img-responsive"}

### In Alpha

GKE On-Prem is current in a closed Alpha, but is already running on client sites.
To request early access sign-up [here](https://cloud.google.com/gke-on-prem/).

### Distribution

This distribution of Kubernetes is first available for installations on
VMWare's VSphere platform.  Bare Metal and OpenStack support were not talked about.
Not much was talked about at a low technical level.

### Single Pane of Glass

Another exciting feature for GKE On-Prem was the controller that allowed any Kubernetes instance to connect into GCP's Kubernetes console.  A demo took a Minikube installation of K8s and connected it to the GCP console.  From there you have all of the functionality to view and monitor any Kubernetes instance.
The tech details were not fully shared, but if you have a Kubernetes cluster that has an accessible control plane API, then the GCP console can communicate with it.
This capability is incredibly powerful.  Have a kops install on AWS, a GKE On-Prem install, and multiple GKE Clusters.  Login to the GCP console and one pane of glass to rule them all!

The amusing thing is that they wanted to connect a GKE On-Prem install running on VSphere for the demo.  They could not get a public IP, so they just used MiniKube.  Frankly, I think the demo at #GoogleNext2018 was far more amazing connecting MiniKube.

## What is Next

I cannot wait to use GKE On-Prem and put it through its paces.  Even the Googlers that I talked to were surprised at their clients results.  Their clients had tried different distributions in their data centers, and GKE On-Prem was not falling as others did.

__Disclaimer:__ I work with Google Professional Services.  Google does not sponsor this blog post.
