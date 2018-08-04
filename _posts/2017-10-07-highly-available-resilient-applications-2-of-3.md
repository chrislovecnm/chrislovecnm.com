---
title: Highly Available Resilient Applications in Kubernetes 2 of 3
description: Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.
header: Highly Available Resilient Applications in Kubernetes 2 of 3
categories: kubernetes best-practices
---

This is the second in 3 that outlines Highly Available (HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.

# Topics Coverred

- Use [Deployment](#deployments) Controller Manifests for Microservice, and use [StatefulSets](#StatefulSets) only if you need their features.
- [Jobs](#jobs) and [DaemonSets](#daemonsets) do not provide out of the box HA, but fill some use cases.
- [Persistent Volumes](#persistent-volumes) are the way to save and make data persitent.

## Kubernetes Controller Manifests

In order have a container hosted on a Kubernetes cluster its API has various
types defined.  These types can be described a Controller Manifests.  Manifests are defined in YAML and JSON
documents.  Specific controller types provide to create HA applications, different types of manifests are
recommended.

1. [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) - declarative packaging for Pods and ReplicaSets.
1. [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) - a Controller that provides multiple guarantees; unique identity, ordering of deployment, and storage.

Other controller types offer some features that can be used to create an application that is HA, but using these types is far more complex.

1. [Job](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/) - one or more Pods that run a specified number of those Pods and successfully terminate.
1. [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) - ensures a single Pod runs on a node as filtered by scheduling rules.

The following controller manifests do not follow HA patterns, or have been replaced by newer manifest types.

1. [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) - the packaging of one of more containers deployed to a single node.
1. [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) - used by Deployment
1. [Replication Controller](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) - replaced by deployments


All the above controller types provide the basic kubernetes functionality of restarting
failed containers.  Restarting a failed container is the most basic pattern that assists
with HA resiliency.

### Deployments

When deploying a stateless microservice, a Deployment is typically the recommended Controller.
This manifest includes the capability to have multiple pods that are deployed
across multiple nodes.  When a Deployment is paired with a Kubernetes Service, the containers within the Deployment are load-balanced with an internal or external endpoint.
When clients communicate with that endpoint, and an event such as a cloud AZ failure occurs, application service is not degraded.

Features include:

1. Multiple replicas of Pods within a deployment
1. Capability for a rolling update of the deployment
1. Ability to roll back a deployment
1. Scale the numbers of a deployment replicas as needed

Two key things must be observed when using deployments: use more than one replica,
create for an application that can function with more than one replica.  Not following these principles only provides the capability for Pod restarts.

The Kubernetes API and `kubectl` the capability for both rolling-updates
and rollbacks.  Deployment’s rollout is triggered when Deployment’s Pod template is updated.

#### Example

A very simple Deployment with an internal service.

{% include code.html language="yaml" file="deployment.yaml"  %}

The following command will upgrade the above deployment to use nginx 1.9.1.

{% highlight shell %}
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated
{% endhighlight %}

Immediately after that command the deployment will be rolled, replacing
each replica with a new Pod running nginx 1.9.1.  This rollout does not
cause service loss.

Deployments can also be rolled backed.  For example if the wrong image
is set:

{% highlight shell %}
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
deployment "nginx-deployment" image updated
{% endhighlight %}

The new pods will be stuck in crash loop, but will not impact the service
level of the deployment.  The deployment status is available with the
following command.

{% highlight shell %}
$ kubectl rollout status deployments nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
{% endhighlight %}

In order to rollback to the previous version the following command is
executed:

{% highlight shell %}
$ kubectl rollout undo deployment/nginx-deployment
deployment "nginx-deployment" rolled back
{% endhighlight %}

By default, two previous Deployment’s rollout version are kept in Kubernetes. Any
version that still exist in Kuberentes can be deployed.  The revision
history is accessible via the API.

{% highlight shell %}
$ kubectl rollout history deployment/nginx-deployment
{% endhighlight %}

Deployments are the primary mechanism that is recommended for HA applications, include when hosting
a stateless application inside of Kubernetes.  At times Deployments have the sufficient capability to run stateful applications inside of Kubernetes, otherwise, the next section covers StatefulSets, developed with features required by some distributed stateful applications.


#### Deployment Documentation

More documentation about Deployments is availible [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### StatefulSets

Creating resilient stateless applications vs creating resilient stateful applications
is very different.  Deployments can be used with stateful applications, but often
 HA distributed stateful applications have certain requirements that are not met by
 Deployments.

StatefulSets provide the following unique features.

1. A stable hostname, available to clients in DNS. The number is based on the StatefulSet name and starts at zero. For example cassandra-0.
1. An ordinal index of Pods. 0, 1, 2, 3, etc.
1. Stable storage linked to the ordinal and hostname of the Pod.
1. Peer discovery is available via DNS. For example, with Cassandra, the names of the peers are known before the Pods creation.
1. Startup and Teardown ordering. Which numbered Pod is going to be created next is known, and which Pod will be destroyed upon reducing the Set size. This feature is useful for such admin tasks as draining data from a Pod when reducing the size of a cluster.

Only choose StatefulSets when one of the above requirement is needed to run an application hosted inside
of Kubernetes.  Many applications such as Kafka, Elastic, Zookeeper, and Cassandra
require one or more of the above capabilities.

As with Deployments, scaling is provided by StatefulSets; but at this time rolling update and rollbacks are
not supported.

#### Example

The following example is for a Cassandra cluster hosted as a StatefulSet.

{% include code.html language="yaml" file="statefulset.yaml"  %}

#### StatefulSets Documentation

More documentation about StatefulSets is availible [here](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).

### Jobs

A Job is a controller that only has HA replay and restart patterns.  This manifest can
be used within Kubernetes, but it is a more complex model to deploy as a resilient application.  This controller does lend
itself well towards queue workers in CQRS.  The application containers handle failure
cases itself, rather than relying on the provided Kuberenetes platform providing fault
tolerance and recovery.

This controller consists of the Pod(s) which run a specified number of them successfully. A simple case is to create one Job to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted, and restarts are controllable within a configurable duration. A Job can also run multiple Pods in parallel.

#### Job Termination and Cleanup

When a Job’s Pods are fails, the Job will keep creating new Pods forever, by default. Retrying forever can be a useful pattern. If an external dependency of the Job’s Pods is missing, then the Job will keep trying to complete.

However, in other cases where a program should not retry forever, the capability exists to set a deadline on the job.  See the below example.

The spec.activeDeadlineSeconds field of the job to some seconds controls cleanup. When the deadline passes, the job will have status with reason: DeadlineExceeded. No more Pods will be created, and existing Pods will be deleted.

#### Example

An example of a simple job running a Perl command.

{% include code.html language="yaml" file="job.yaml"  %}

#### Jobs Documentation

More documentation about Jobs is available
[here](https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/).

### DaemonSets

Another controller that is very useful, but does not lend itself toward high availability,
is a DaemonSet.  Deploy of HA application as DaemonSets is very complex.

This manifest ensures that a Pod runs on all or some nodes.

Some typical uses of a DaemonSet are:
- Running a CNI networking provider
- Running a logs collection daemon
- Running a node monitoring daemon

A Daemonset is analogous to have a Systemd Unit hosted on every node. Just as with Jobs, to be HA, applications hosted as a DaemonSets must include functionality
to handling failovers and restarts.  Often, replay logs and other patterns are used.

#### Example

The following is a Daemonset for Weave.

{% include code.html language="yaml" file="daemonset.yaml"  %}

#### DaemonSets Documentation

More documentation about DaemonSets is available [here](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

## Kuberenetes Storage

Storage in a container is ephemeral when the storage is not a mounted volume.  Within
Kubernetes some storage types are transient: `emptyDir` and `hostPath`.  Both of those
storage types do not maintain persistence between Pod or node restarts or failures.

Kubernetes volumes, such as `awsElasticBlockStore` has an explicit lifetime. Volumes can outlive container restarts, and data persists between container restarts. Destroying a Pod can cause data loss.  On of the features of StatefulSets is maintaining data persistence upon Pod is deletion.

### EBS Based Storage

An `awsElasticBlockStore` volume mounts an EBS Volume.

There are some limitations when using an `awsElasticBlockStore volume`:

- Nodes need to be in the same region and availability zone as the EBS volume
- EBS only supports a single EC2 instance mounting a volume

These limitations must be kept in mind when designing for such occurrences as AZ failures.

### Persistent Volumes

The PersistentVolume subsystem provides an API that abstracts storage provision and consumption.  A PersistentVolume (PV) is a piece of storage in a K8s cluster, while a PersistentVolumeClaim (PVC) is a request for storage for a controller.  The third component included is a StorageClass, which represents various classes of storage.

Persistent volumes and the underlying storage must exist before a controller is associated
to the storage.

#### Persistent Volume Claim Templates

A feature of the StatefulSet controller, VolumeClaimTemplates is a list of storage claims that Pods reference.
A StatefulSet controller maps network identities, persistent volumes, to claims in a way that maintains the identity of Pod.
When creating a StatefulSet, with a template, in a Kubernetes cluster, the persistent volumes are automatically created.
The Cassandra example included in this document utilized a volume claim template.

#### Documentation

More documentation about storage is available [here](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

Next posts will continue on to the topics of probes and affinity rules.
