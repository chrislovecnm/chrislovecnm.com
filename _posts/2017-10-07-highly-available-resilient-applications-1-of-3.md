---
title: Highly Available Resilient Applications in Kubernetes 1 of 3
description: Highly Available(HA) and application resilience best practices for running a custom or third party application hosted inside a Kubernetes (K8s) cluster.
header: Highly Available Resilient Applications in Kubernetes 1 of 3
date:   2017-10-06 22:00:40 -0600
categories: kubernetes best-practices
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
- Use [Deployment]({% post_url 2017-10-07-highly-available-resilient-applications-2-of-3 %}#deployments) Controller Manifests for Microservice, and use [StatefulSets]({% post_url 2017-10-07-highly-available-resilient-applications-2-of-3 %}#StatefulSets) only if you need there features.
- [Jobs]({% post_url 2017-10-07-highly-available-resilient-applications-2-of-3 %}#jobs) and [DaemonSets]({% post_url 2017-10-07-highly-available-resilient-applications-2-of-3 %}#daemonsets) do not provide out of the box HA, but fill some use cases.
- [Persistent Volumes]({% post_url 2017-10-07-highly-available-resilient-applications-2-of-3 %}#persistent-volumes) are the way to save a make data persitent.

### Covered in Part Three
- Use Liveness and Ready [Probes]({% post_url 2017-10-07-highly-available-resilient-applications-3-of-3 %}#probes). Design your application to use and support
them.
- Use [Affinity and Anti-Affinity Selectors]({% post_url 2017-10-07-highly-available-resilient-applications-3-of-3 %}#affinity-and-anti-affinity-selectors) if pods need
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
container.  More detailed information about [Container Lifecycle Hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/
) is
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

There is a well known [process ID 1 problem](
https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/)
that can add complexity to handling signals within containers.  Depending on the
executable used from a Dockers ENTRYPOINT, that problem can cause complexities.

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
within containers.  One such tool that used within Trebuchet is
[dumb-init](https://github.com/Yelp/dumb-init).
[Yelp](https://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html)
open sourced this a small C-based binary to solved the two problems listed
above, and more:

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

Next posts will cover manifest types and controlling scheduling.
