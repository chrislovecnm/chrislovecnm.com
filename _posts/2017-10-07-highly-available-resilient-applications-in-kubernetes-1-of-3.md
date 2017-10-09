---
title: Highly Available Resilient Applications in Kubernetes 1 of 3
description: Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.
header: Highly Available Resilient Applications in Kubernetes 1 of 3
---


This is the first in 3 that outlines Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.  High Availability and resilience allow us to handle; infrastructure and applications failures, cloud outages where the Kuberentes cluster is still functional, rolling updates of K8s, and rolling updates of applications.  One of the guiding principles of Kubernetes is HA fault tolerance, but Kubernetes provides a platform to build applications that meet HA SLAs, it does not make applications fault tolerant.

Whenever I think about architecture I follow a simple thought process.  What do I need
to do to design and deploy and application, so that I am not woke up at 2am in the
morning because of a page goes off.  And if I am woken up at 2am, how can I setup
a system that will failover and recover itself by the time that I log into the Kubernetes
cluster.

## TLDR;

These are not nice to haves but must haves for designing and deploying an
application hosted in Kubernetes.

- Make sure your application stops gracefully when it gets a `SIGTERM` signal. See [Gracefully handling container stop Signal Handling within Kubernetes](#gracefully-handling-container-stop-signal-handling-within-kubernetes).
- Use and `ENTRYPOINT` with [dumb-init](#dumb-init) so that signals are passed properly to your binary.
- Use Pre or Post stop hooks if your binary needs more TLC to start or stop gracefully.

### Covered in Part Two
- Use [Deployment](#deployments) Controller Manifests for Microservice, and use [StatefulSets](#StatefulSets) only if you need there features.
- [Jobs](#jobs) and [DaemonSets](#daemonsets) do not provide out of the box HA, but fill some use cases.
- [Persistent Volumes](#persistent-volumes) are the way to save a make data persitent.

### Covered in Part Three
- Use Liveness and Ready [Probes](#probes). Design your application to use and support
them.
- Use [Affinity and Anti-Affinity Selectors](#affinity-and-anti-affinity-selectors) if pods need
to be ditributed across nodes.

## Application Lifecycle Within Kubernetes

A container, which hosts an application, can be made aware of events in its
lifecycle.  This information is essential for a hosted application to be alerted
that it started or notified that it is stopping.  Within various scenarios,
including a pod eviction from a Kubernetes node.  Another such scenic is when a
Kubernetes node is drained, before destroying that node.

When an event occurs, kubelet calls into any registered container hook for that
event.  The hook calls are synchronous in the processing of the container.  This
means for a pre-start hook the container entry point, and the hook will fire
asynchronously.  Hooks also impact the state of a container within the
Kubernetes system.  For example, if a `PostStart` hook fails, the container will
not reach "running" state.

### Container Hooks

Hooks execute as either an HTTP request or an execution of a command within the
container.  More detailed information about [Container Lifecycle Hooks][1] is
found via the provided link.

#### PostStart Hook

This hook fires after a container creation, and often runs at the same time as a containers entry point.  Since this is an asynchronous call when the hook runs the timing is not guaranteed.

#### PreStop Hook

This hook executes before a container termination, while the PID is still running.  `PreStop` hook event is blocking and completes before the call to delete the container is sent to the Docker daemon.

### Container Hook Use

To make an application more resilient application tasks may need to be completed, or some runtime executables may need some help with signal handling to stop.  Often when using Java JVM, a container will not handle a shutdown gracefully.

Including the following example, `PreStop` to JVM based containers is often helpful.  This example lives within the contain section of a Kubernetes manifest.

{% highlight yaml linenos %}
lifecycle:
  preStop:
    exec:
      command:
        - /bin/bash
          - -c
            - PID=`pidof java` && kill -SIGTERM $PID && while ps -p $PID > /dev/null; do sleep 1; done;
{% endhighlight %}

Other applications such as Nginx will stop when they receive an `SIGTERM` signal.  To gracefully stop Nginx use the following `preStop` hook.

{% highlight yaml linenos %}
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        lifecycle:
          preStop:
            exec:
              # use this command to gracefully shutdown
              command: ["/usr/sbin/nginx","-s","quit"]
{% endhighlight %}

### Gracefully handling container stop Signal Handling within Kubernetes

When Kuberentes shuts down a container, two different Unix Signals run: `SIGTERM` and `SIGKILL`. An example of the workflow to stop a pod and its container(s).

1. The Kubernetes API receives a call command to delete a container or pod.
2. Default grace period of 30s starts unless otherwise configured.
3. Pod status is set to "Terminating."
4. Kubelet starts the pod shutdown process.
5. If a `preStop` hook exists it executes.
7. The processes in the Pod's containers are sent the SIGTERM signal
8. If the processes are still running after the default grace period, an SIGKILL signal given to the processes.
9. Kubelet updates K8s API removing the pod when kubelet finished deleting the pod and its container(s).

The application must receive the correct signals and handle those flags.  Moreover, properly designed, properly behaving, and appropriately deploy applications should not get to the point where an SIGKILL signal is not needed.

#### Complexity of Signal Handling in Containers

There is a well known [process ID 1 problem][2] that can add complexity to
handling signals within containers.  Depending on the executable used from a
Dockers ENTRYPOINT, that problem can cause complexities.

_TLDR;_

> Process ID 1, or PID 1, is a special process id that the kernel
reserves for init scripts.  Because init scripts are not used within containers,
having an applications PID running as PID 1 can cause unexpected and
obscure-looking issues.

Moreover, various implementations of the UNIX shell, `/bin/sh`, do not pass
signals to their child processes.  For instance, the default implementation of
shell in the alpine base container does not send interupts to its child
processes.

A simple solution is to use a binary that acts as a signal proxy and starts a
child process as PID 2 inside a container.

##### dumb-init

Various binaries exist that assist with PID management and signal proxying
within containers.  One such tool that used within Trebuchet is [dumb-init][3].
[Yelp][4] open sourced this a small C-based binary to solved the two problems
listed above, and more:

1. `dumb-init` starts as PID 1 and then start a container application as PID 2.
2. `dumb-init` proxies any UNIX signals, such as SIGTERM, to its child process PID 2.
3. `dumb-init` reaps any zombie process created.

One of our base Docker images `dumb-init` contains the `ENTRYPOINT` for
dumb-init.

{% highlight docker linenos %}
ENTRYPOINT ["/sbin/dumb-init"]
{% endhighlight %}

If your application uses the above container, add a `CMD` reference in
applications Dockerfile.

{% highlight docker linenos %}
FROM "our-repo:dumb-init:0.ourversion"
CMD ["/my-app"]
{% endhighlight %}

When the above container the executes its `ENTRYPOINT`, the `CMD` runs as an
argument.

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
