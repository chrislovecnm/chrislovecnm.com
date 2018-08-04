---
title: Highly Available Resilient Applications in Kubernetes 3 of 3
description: Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.
header: Highly Available Resilient Applications in Kubernetes 3 of 3
categories: kubernetes best-practices
---

This is the post that outlines Highly Available (HA) and application resilience best practices for running a custom, or third party application hosted inside a Kubernetes (K8s) cluster.

# Topics Covered
- Use Liveness and Ready [Probes](#probes). Design your application to use and support
them.
- Use [Affinity and Anti-Affinity Selectors](#affinity-and-anti-affinity-selectors) if pods need
to be distributed across nodes.


## Pod Lifecycle and Probes

Kubernetes provides two different types of probes to assist in managing an applications Readiness and Liveness, kubelet diagnostics actions.

### Probes

Two kinds of probes exist in Kubernetes:

1. `readinessProbe`: Indicates whether the Container is ready to service requests. If the readiness probe fails, the endpoints controller does not add the Pod’s IP address to all Services that match the Pod.
2. `livenessProbe`: A validation whether the Container is running. If the liveness probe fails, kubelet kills the Container, and the Container is subjected to its restart policy.

When using both probes, the `livenessProbe` is not executed by Kubelet,
until after the `readinessProbe` passes.

Three different implementations of probes exist:

1. `HTTPGetAction`: Diagnostic HTTP call to a container that expects a 200 or a 400.
2. `ExecAction`: Executes a command inside the container, and expects an `exit(0)` for success.
3. `TCPSocketAction`: Creates a socket connection on a specified port, and succeeds if the socket connects.

Each of the probes has one of three results; Success, Failure and Unknown.  More details on probes are found in the Kubernetes [documentation](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/).

#### When To Use Or Not Use Probes

If an application does not have a manner to check that it is performing correctly, then an application must exist on its own when it encounters a fatal error or becomes unhealthy.  It is a good practice to add an external contact point, because often applications do not know when they become unresponsive.  If a program does not have an appropriate monitoring point, then using a liveness probe is impracticable.

When an application has a warm-up or startup process and is not immediately open for business, use a ready probe.  This is often used by applications that deployed as stateful sets, where only one of the members of an HA application should be starting at the same time.  A Pod will not receive any service based traffic until the ready probe passes.

TLDR;

> Design applications to have a monitoring point for Liveness probes, and if needed, a Readyprobe.

#### Examples

The following example is for elastic:

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

Note: This is for the entire pod, not a single container.  The container restarts, but not all the containers in a Pod.

Different Kubernetes manifests support specific policies.  Jobs support OnFailure or Never, while Deployments support all three.  A Daemon Set has a policy of Always.

## Pod Scheduling on Kubernetes

The Kubernetes scheduler, which runs as a set of HA Pods, determines on which node
a Pod can fit the best on, dependant on configurable policies.  The generic algorithm for
Pod scheduling follows:

1. Filter the nodes: run each node through a list of predicates based on any constraints or
requirements specified by the Pods.
2. The filtered list of nodes is prioritized: score each node depend on node weighting
and proper Pod fit.
3. Based on the order list above schedule the Pod on the best fit node.

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

With the generic scheduling algorithm for Pods, the scheduler makes its best attempt
to schedule a Deployments replicas fairly evenly around nodes.  But that is not
contractually promised.

In certain cases, such as running an HA distributed application like Zookeeper, Kafka, or Elastic, it is desirable not to deploy Pods of the same type to the same node, and spread evenly across availability zones within EC2.

### Affinity and Anti-Affinity Selectors

With certain applications, it is necessary to provide the scheduler with filtering
information to creating affinity or anti-affinity rules, for which nodes Pods from an application are deployed upon.
Kubernetes allows for users to control both `nodeAffinity` and `podAffinity`. These are fields defined in a manifests Pod metadata section. Affinity and Anti-Affinity provide
the following capabilities:

- Create a filter(s) where the scheduler runs a Pod based on which other Pods are, or are not running on a node.
- A request without requiring that a Pod runs on a node.
- Specify a set of allowable values instead of a single value requirement.

| AFFINITY SELECTOR    | REQUIREMENTS MET | REQUIREMENTS NOT MET    | REQUIREMENTS LOST |
|-------------------| -----------------|----------------------|-------------------|
| preferredDuringSchedulingIgnoredDuringExecution |    Runs| Runs    | Keeps Running |
| requiredDuringSchedulingIgnoredDuringExecution    | Runs    | Fails    | Keeps Running |

Affinity rules give weight to where a Pod deploys, while Anti-Affinity rules
provide a weight to direct where a Pod is not deployed.  When developing and deploying fault tolerant applications, it is often more important to ensure that Pods do not live on the same node.  Anti-Affinity Selectors provide that exact functionality.

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

As previously mentioned the scheduler, without specific filters, spreads Pods across
nodes.  When specific selectors are added to the scheduler filters, the filters do impact the speed of a deployment of a Pod.  Exact numbers on the impact of scaling are not available at this time, but this impact is a known issue.  The recommendation is to use the selector only
when needed, and most often with applications that have state and maintain data persistence.
