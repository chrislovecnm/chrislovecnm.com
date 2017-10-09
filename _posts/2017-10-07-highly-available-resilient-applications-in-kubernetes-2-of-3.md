---
title: Highly Available Resilient Applications in Kubernetes 2 of 3
description: Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.
header: Highly Available Resilient Applications in Kubernetes 2 of 3
---

This is the second in 3 that outlines Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.

# Topics Coverred

- Use [Deployment](#deployments) Controller Manifests for Microservice, and use [StatefulSets](#StatefulSets) only if you need there features.
- [Jobs](#jobs) and [DaemonSets](#daemonsets) do not provide out of the box HA, but fill some use cases.
- [Persistent Volumes](#persistent-volumes) are the way to save a make data persitent.

## Kubernetes Controller Manifests

In order have a container hosted on a Kubernetes cluster its API has various
types defined.  These types can be described a Controller Manifests.  Manifests are defined in YAML and JSON
documents.  Specific controller types provide to create HA applications different types of manifests are
recommended.

1. [Deployments][6] - declarative packaging for Pods and ReplicaSets.
1. [StatefulSets][7] - a Controller that provides multiple guarantees: unique identity, ordering of deployment, and storage.

Other controller types offer some features that can be used to create an application that is HA, but using these types is far more complex.

1. [Job][9] - one or more pods that run specified number of those Pods and successfully terminate.
1. [DaemonSet][8] - ensures a single Pod runs on a node as filtered by scheduling rules.

The following controller manifests do not follow HA patterns or have been replaced by newer manifest types.

1. [Pod][10] - the packaging of one of more containers deployed to a single node.
1. [ReplicaSet][11] - replaced by Deployments
1. [ReplicationController] - replaced by Deployments

All the above controller types provide the basic kubernetes functionality of restarting
failed containers.  Restarting a failed container is the most basic pattern that assists
with HA resiliency.

### Deployments

When deploying a stateless microservice, a Deployment is typically the recommended Controller.
This manifest includes the capability to have multiple pods that are deployed
across multiple nodes.  When a Deployment is pair with a Kubernetes Service the containers within the Deployment are load balanced with an internal or external endpoint.
When clients communicate with that endpoint, and an event such as a cloud AZ failure occurs, application service is not degraded.

Features include:

1. Multiple replicas of pods within a deployment
1. Capability for a rolling update of the deployment
1. Ability to roll back a deployment
1. Scale the numbers of a deployment replicas as needed

Two key things must be observed when using deployments: use more than one replica,
create or an application that can function with more than one replica.  Not following these principles only provides the capability for Pod restarts.

The Kubernetes API and `kubectl` the capability for both rolling-updates
and rollbacks.  Deployment’s rollout is triggered when Deployment’s pod template is updated.

#### Example

A very simple Deployment with an internal service.

{% include code.html language="yaml" file="deployment.yaml"  %}

The following command will upgrade the above deployment to use nginx 1.9.1.

{% highlight shell %}
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated
{% endhighlight %}

Immediately after that command the deployment will be rolled, replacing
each replica with a new pod running nginx 1.9.1.  This rollout does not
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

More documentation about Deployments is availible [here][6].

### StatefulSets

Creating resilient stateless applications vs creating resilient stateful applications
is very different.  Deployments can be used with stateful applications but often
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
require one of more of the above capabilities.

As with Deployments scaling is provided by StatefulSets, but at this time rolling update and rollbacks are
not supported.

#### Example

The following example is for a Cassandra cluster hosted as a StatefulSet.

{% include code.html language="yaml" file="statefulset.yaml"  %}

#### StatefulSets Documentation

More documentation about StatefulSets is availible [here][7].

### Jobs

A Job is a controller that only has HA replay and restart patterns.  This manifest can
be used within Kubernetes but it is a more complex model to deploy as a resilient application.  This controller does lend
itself well towards queue workers in CQRS.  The application containers handle failure
cases itself, rather than relying on the provided Kuberenetes platform providing fault
tolerance and recovery.

This controller consists of the pod(s) which run a specified number of them successfully. A simple case is to create one Job to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted, and restarts are controllable within a configurable duration. A Job can also run multiple Pods in parallel.

#### Job Termination and Cleanup

When a Job’s pods are fails, the Job will keep creating new Pods forever, by default. Retrying forever can be a useful pattern. If an external dependency of the Job’s pods is missing, then the Job will keep trying to complete.

However, in other cases where a program should not retry forever, the capability exists to set a deadline on the job.  See the below example.

The spec.activeDeadlineSeconds field of the job to some seconds controls cleanup. When the deadline passes, the job will have status with reason: DeadlineExceeded. No more pods will be created, and existing pods will be deleted.

#### Example

An example of a simple job running a Perl command.

{% include code.html language="yaml" file="job.yaml"  %}

#### Jobs Documentation

More documentation about Jobs is available [here][9].

### DaemonSets

Another controller that is very useful, but does not lend itself toward high availability
is a DaemonSet.  Deploy of HA application as DaemonSets is very complex.

This manifest ensures that a pod runs on all or some nodes.

Some typical uses of a DaemonSet are:
- running a CNI networking provider
- running a logs collection daemon
- running a node monitoring daemon

A Daemonset is analogous to have a Systemd Unit hosted on every node. Just as with Jobs, to be HA, applications hosted as a DaemonSets must include functionality
to handling failovers and restarts.  Often replay logs and other patterns are used.

#### Example

The following is a Daemonset for Weave.

{% include code.html language="yaml" file="daemonset.yaml"  %}

#### DaemonSets Documentation

More documentation about DaemonSets is availible [here][8].

## Kuberenetes Storage

Storage in a container is ephemeral when the storage is not a mounted volume.  Within
Kubernetes some storage types are transient: `emptyDir` and `hostPath`.  Both of those
storage types do not maintain persistence between pod or node restarts or failures.

Kubernetes volumes, such as `awsElasticBlockStore` has an explicit lifetime. Volumes can outlive container restarts, and data persists between container restarts. Destroying a Pod can cause data loss.  On of the features of StatefulSets is maintaining data persistence upon pod is deletion.

### EBS based Storage

An `awsElasticBlockStore` volume mounts an EBS Volume.

There are some limitations when using an `awsElasticBlockStore volume`:

- nodes need to be in the same region and availability zone as the EBS volume
- EBS only supports a single EC2 instance mounting a volume

These limitations must be kept in mind when designing for such occurrences as AZ failures.

### Persistent Volumes

The PersistentVolume subsystem provides an API that abstracts storage provision and consumption.  A PersistentVolume (PV) is a piece of storage in a K8s cluster, while a PersistentVolumeClaim (PVC) is a request for storage for a controller.  The third component included is a StorageClass, which represents various classes of storage.

Persistent volumes and the underlying storage must exist before a controller is associated
to the storage.

#### Persistent Volume Claim Templates

A feature of the StatefulSet controller, VolumeClaimTemplates is a list of storage claims that pods reference.
A StatefulSet controller maps network identities, persistent volumes, to claims in a way that maintains the identity of a pod.
When creating a StatefulSet, with a template, in a Kubernetes cluster, the persistent volumes are automatically created.
The Cassandra example included in this document utilized a volume claim template.

#### Documentation

More documentation about storage is availible [here][13].

## Pod Lifecycle and Probes

Kubernetes provides two different types of probes to assist in managing an applications Readiness and Liveness, kubelet diagnostics actions.

### Probes

Two kinds of probes exist in Kubernetes.

1. `readinessProbe`: Indicates whether the Container is ready to service requests. If the readiness probe fails, the endpoints controller does not add the Pod’s IP address to all Services that match the Pod.
2. `livenessProbe`: A validation whether the Container is running. If the liveness probe fails, kubelet kills the Container, and the Container is subjected to its restart policy.

When using a both probes, the `livenessProbe` is not executed by Kubelet,
until after the `readinessProbe` passes.

Three different implementations of probes exist:

1. `HTTPGetAction`: Diagnostic HTTP call to a container that expects a 200 or a 400.
2. `ExecAction`: Executes a command inside the container, and expects an `exit(0)` for success.
3. `TCPSocketAction`: Creates a socket connection on a specified port, and succeeds if the socket connects.

Each of the probes has one of three results; Success, Failure and Unknown.  More details on probes are found in the Kubernetes [documentation][5].

#### When to use or not use probes

If an application does not have a manner to check that it is performing correctly, then an application must exist on its own when it encounters a fatal error or becomes unhealthy.  It is a good practice to add an external contact point because often applications do not know when they become unresponsive.  If an program does not have an appropriate monitoring point then using a liveness probe is impracticable.

When an application has a warm-up or startup process and is not immediately open for business, use a ready probe.  This is often used by applications that deployed as stateful sets, where only one of the members of an HA application should be starting at the same time.  A pod will not receive any service based traffic until the ready probe passes.

TLDR;

> Design applications to have a monitoring point for Liveness probes, and if needed a Readyprobe.

#### Examples

The following example is for elastic.

{% highlight yaml linenos %}
          livenessProbe:
            exec:
              command:
                - /bin/bash
                - -c
                - 'if [ "$(curl -sf 0:9200/_cat/nodes?h=ip | grep $POD_IP)" != "$POD_IP" ]; then exit 1; fi'
            initialDelaySeconds: 120
            timeoutSeconds: 5
          readinessProbe:
            exec:
              command:
                - /bin/bash
                - -c
                - 'if [ "$(curl -sf 0:9200/_cat/nodes?h=ip | grep $POD_IP)" != "$POD_IP" ]; then exit 1; fi'
            initialDelaySeconds: 30
            timeoutSeconds: 5
{% endhighlight %}

Each container within a Pod can contain a probe.  Or a sidecar can run inside a Pod that provides monitoring functionality.  The above YAML is part of the `containers` section in a Pod manifest.

### Restart Policy

The Pod configuration `RestartPolicy` interacts with a containers `livenessProbe`.  When the `livenessProbe` fails, and kubelet stops the container, kubelet honors a containers `RestartPolicy` next.  The types of pods `RestartPolicy`:

1. Always
2. OnFailure
3. Never

Note: This is for the entire pod, not a single container.  The container restarts, but not all the containers in a pod.

Different Kubernetes manifests support specific policies.  Jobs support OnFailure or Never, while Deployments support all three.  A Daemon Set has a policy of Always.

## Pod Scheduling on Kubernetes

The Kubernetes scheduler, which runs as a set of HA pods, determines on which node
a pod can fit the best on, dependant on configurable policies.  The generic algorithm for
pod scheduling follows:

1. Filter the nodes: run each node through a list of predicates based on any constraints or
requirements specified by the pods.
2. The filtered list of nodes is prioritized: score each node depend on node weighting
and proper pod fit.
3. Based on the order list above schedule the pod on the best fit node.

### Scheduling Impact on Application High Availability

The resilience of an application can be affected by various factors that are influenced by
scheduling.  These factors include:

1. Which AZ a Node live in
2. EBS volumes are unique to an AZ
3. When multiple HA Pods are scheduled to the same node

These factors can impact an applications HA capability during:

1. AZ outages
2. Node / Node software outages
3. Pod eviction
4. Inability to mount an EBS volume due to lack of node headroom in
the AZ where the EBS volume lives.

With the generic scheduling algorithm for pods, the scheduler makes its best attempt
to schedule a Deployments replicas fairly evenly around nodes.  But that is not
contractually promised.

In certain cases, such as running an HA distributed application such as Zookeeper, Kafka, or Elastic, it is desirable not to deploy Pods of the same type are to the same node and spread evenly across availability zones within EC2.

### Affinity and Anti-Affinity Selectors

With certain applications, it is necessary to provide the scheduler with filtering
information to creating affinity or anti-affinity rules for which nodes pods from an application are deployed upon.
Kubernetes allows for users to control both `nodeAffinity` and `podAffinity`. These are fields defined in a manifests Pod metadata section. Affinity and Anti-Affinity provide
the following capabilities:

- Create a filter(s) where the scheduler runs a pod based on which other pods are or are not running on a node.
- A request without requiring that a pod runs on a node.
- Specify a set of allowable values instead of a single value requirement.

| AFFINITY SELECTOR    | REQUIREMENTS MET | REQUIREMENTS NOT MET    | REQUIREMENTS LOST |
|-------------------| -----------------|----------------------|-------------------|
| preferredDuringSchedulingIgnoredDuringExecution |    Runs| Runs    | Keeps Running |
| requiredDuringSchedulingIgnoredDuringExecution    | Runs    | Fails    | Keeps Running |

Affinity rules give weight to where a pod deploys, while Anti-Affinity rules
provide a weight to direct where a pod is not deployed.  When developing and deploying fault tolerant applications, it is often more important to ensure that pods do not live on the same node.  Anti-Affinity Selectors provide that exact functionality.

### Anti-Affinity Select Example

The following YAML utilizes Anti-Affinity selectors that request that the scheduler
attempts to not schedule the same pod type on the same node.  The `matchExpressions.'
is keyed by the `app` label.

{% highlight yaml linenos %}
spec:
  replicas: 3
  template:
    metadata:
      labels:
        component: "elasticsearch"
        subcomponent: "client"
        app: "elasticsearch-client"
      annotations:
        scheduler.alpha.kubernetes.io/affinity: >
          {
            "podAntiAffinity": {
              "preferredDuringSchedulingIgnoredDuringExecution": [{
                "labelSelector": {
                  "matchExpressions": [
                    { "key": "app", "operator": "In", "values": ["elasticsearch-client"] }
                  ]
                },
                "topologyKey": "kubernetes.io/hostname",
                "weight": 1
              }]
            }
          }
{% endhighlight %}

Affinity rules API in Kubernetes 1.6 has been changed.  The definition for
the selectors has moved into the `PodSpec`.

{% include code.html language="yaml" file="nginx-pod-affinity.yaml"  %}

### Shortcomings of Affinity and Anti-Affinity Selectors

As previously mentioned the scheduler, without specific filters, spreads pods across
nodes.  When specific selectors are added to the scheduler filters, the filtersdo impact the speed of a deployment of a pod.  Exact numbers on the impact of scaling are not available at this time, but this impact is a known issue.  The recommendation is to use the selector only
when needed, and most often with applications that have state and maintain data persistence.

[1]: [https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/]
[2]: [https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/]
[3]: [https://github.com/Yelp/dumb-init]
[4]: [https://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html]
[5]: [https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/]
[6]: [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/]
[7]: [https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/]
[8]: [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/]
[9]: [https://kubernetes.io/docs/concepts/jobs/run-to-completion-finite-workloads/]
[10]: [https://kubernetes.io/docs/concepts/workloads/pods/pod/]
[11]: [https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/]
[12]: [https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/]
[13]: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/]
