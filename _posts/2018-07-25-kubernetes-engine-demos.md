---
title: Kubernetes Engine Demos
description: What I have been doing for the last months, working with Kubernetes Engine
header: Kubernetes Engine Demos
date:   2018-07-25 06:54:49 PDT
categories: Kubernetes
tags:
  - Kubernetes Engine
  - Kubernetes
  - Demos
---

## Where's Waldo

For the last three months, my GitHub Contributions scoreboard has been looking like an
ugly shade of grey.

![GitHub Scoreboard](/img/github-contributions.png){:class="img-responsive"}

I had one community managers for Kubernetes reach out and ask if I had been
eaten by the all encompasing XYZ.  XYZ being her company.  Yes I have been
working with a world class team with a major client on a fun, and now public,
project.  Thanks to my partners over at ReactiveOps, I was given the opportunity
to work with Google Professional Services Project Helmsman that I can only call
amazingly awesome.

The team that I work with has been working hard on getting various assets ready
for #GoogleNext2018, and some of those assets include Open Source examples using
Kubernetes Engine.


## Open Source with Google Cloud Platform - Project Helmsman

As many of you know, Google is a huge proponent and driver of many Open Source
projects including Kubernetes. The projects that we released on 07/25/2018
include 15 projects covering various topics focusing on using Kuberenetes Engine
on GCP, and connecting Kubernetes Engine on GCP to [Kubernetes Engine
On-Prem](https://cloud.google.com/gke-on-prem/).  This release coincided with
the news that Google was releasing the product Kubernetes Engine On-Prem.

## The Projects

At this point, we have released 15 projects on GitHub that can be all found by
clicking [here](http://bit.ly/kubernetes-engine-demos). In those demos and
tutorials we covered a range of topics broken into various areas; Databases,
Networking, Rolling Updates, Istio, Security, Logging Monitoring and Tracings.

Yes those are a bunch of topics, and there are 15 projects, so this is not a short
post.

## Databases

### GKE Stateful Applications Demo

Leveraging my previous work with Apache Cassandra and Kubernetes, we
implemented a new project that demonstrates a Stateful Application, and frankly,
the complexities of running a Stateful App on Kubernetes Engine.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-stateful-applications-demo)

### Cloud SQL

Running a Cloud SQL Proxy sidecar with a MySQL or Postgres container provides a
seamless way to tie an application into GCP, IAM, and a hosted MySQL or Postgres
Cloud SQL database.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-cloud-sql-postgres-demo)

## Logging, Monitoring, and Tracing

### Monitoring with Stackdriver

This demo has you setup Monitoring and visualization for a Kubernetes Engine
Cluster.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-monitoring-tutorial)

### Logging with Stackdriver

You can use this demo to deploy a sample application to GKE and fowards log
events to Stackdriver logging. Logs are forwards to a Cloud Storage bucket, and
also BigQuery.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-logging-sinks-demo)

### Tracing with Stackdriver

Stackdrivers tracing feature is demonstrated by this project. This application
deploys a tiny Python Flask application that uses Open Census tracing and
integrates into a Pub Sub Topic.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-tracing-demo)

### Monitoring with Datadog

Other third party tools can be used to monitor Kubernetes Engine. This code will
help you deploy a Datadog Daemonset, and send data to their dashboards.  The
Datadog agents will be configured to monitor the nginx workload, and ship
metrics to your own Datadog account.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-datadog-demo)

## Networking

### Connect GCP Networks with VPC Peering

This demo uses deployment manager to multiple GKE clusters, and enables VPC
Peering between the different clusters. Various applications and service
types are deployed to demonstrate different Kubernetes Services.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-networking-demos/tree/master/gke-to-gke-peering)

### Connect GCP Networks with Cloud VPN

Instead of using VPC Peering, this project duplicates the above project using 
a VPN connection.  Clusters in one project will be used as stand-ins for
"on-premise" clusters, and the VPN will demonstrate remote communication between
those clusters, and Kubernetes Engine clusters. 

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-networking-demos/tree/master/gke-to-gke-vpn)

## Rolling Updates

This GitHub repository includes three different patterns that can be used to
upgrade a GKE cluster. The first is the out of the box upgrade built in; the
second two are using patterns that are a great choice for more mission-critical
and stateful applications.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-rolling-updates-demo)

### In Place Rolling Upgrade

This project demonstrates the out of the box upgrade functionality for
Kubernetes Engine.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-rolling-updates-demo/tree/master/in-place-rolling-upgrade)

###  Expand And Contract Upgrade

Using an "Expand and Contract" pattern, this demo has you upgrade a cluster.  The
Expand and Contract Upgrade pattern increases headroom by adding 1 or more new
nodes to the node pool, prior to starting the upgrade. When the upgrade has
completed, the extra nodes are removed.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-rolling-updates-demo/tree/master/expand-contract-upgrade/tree/master/in-place-rolling-upgrade)

###  Blue/Green Upgrade

Using this project you perform a cluster upgrade using a "lift and shift"
strategy otherwise know as blue/green. New nodepool(s) are created, and then
applications are moved off the old nodes, and on to the new nodepool(s).  The
old nodepool(s) are then removed.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-rolling-updates-demo/tree/master/blue-green-upgrade)

## Security

### Network Policies

Network Policies allow fine-grained control and restriction of network
communication, internal and external of a Kubernetes Engine Cluster. You can go
though this demo and provision a cluster, various pods, and configure their
network communication capabilities.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-network-policy-demo)

### Role Based Access Control (RBAC)

This project allows you to assign different permissions to different users and
grant GKE API access to a example application running inside a Kubernetes Engine
Cluster.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-rbac-demo)

### Securing Containerized Applications

You will deploy multiple instances of the same container image with a variety of
security settings to illustrate the use of RBAC, security contexts, and AppArmor
policies.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-application-security-demo)

### Controlling Levels of Privilege

You will use a combination of RBAC, network policies, security contexts, and
AppArmor profiles to enforce appropriate security constraints on three distinct
applications.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-security-scenarios-demo)

## Istio

### Istio with Telemetry

With this project you can deploy Istio, Prometheus, Jaeger and Grafana.  This
demonstrates how to use all four tools together on a GKE cluser.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-istio-telemetry-demo)

### Istio with Mesh Expansion

With Istio you can expand the service mesh, as that is a base capability.  This
demo installs Istio on a GKE cluster, creates a GCE instance with MySQL
installed.  Then the GCE Instance is added to the mesh.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-istio-gce-demo)

### Istio with Mesh Expansion Across Networks

This demo is the same as above, but uses a VPN connection to demonstrate a
mock on-prem connection.

[Go to the demo project](https://github.com/GoogleCloudPlatform/gke-istio-vpn-demo)

## What's Next

We are still busy working on few projects, but mostly looking forward to working
with anyone who would like to help us fix any problems, and improve the
communites OSS demos.  We love to see contributions and issues!

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg1NjQyNjAyM119
-->
