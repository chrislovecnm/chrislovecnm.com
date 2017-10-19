[kops](https://github.com/kubernetes/kops) is an Open Source top level
Kubernetes project.  kops is an abbreviation for Kuberentes Operations. From
the README in the project:

> kops helps you create, destroy, upgrade and maintain production-grade, highly
available, Kubernetes clusters from the command line. AWS (Amazon Web Services)
is currently officially supported, with GCE and VMware vSphere in alpha and
other platforms planned.

Founded by the Kubernetes Team in 2016. [Justin Santa
Barbara](https://github.com/justinsb) said that he did not find kops, but it
found us.  [Mike Danese](https://github.com/mikedanese), [Zach
Loafman](https://github.com/zmerlynn), Justin, and many other people from the
Kubernetes team contributed to the initial creation of kops.

## kops is DevOps for Kubernetes

[Wikipedia's](https://en.wikipedia.org/wiki/DevOps) DevOps Definition

> DevOps (a clipped compound of "development" and "operations") is a software
engineering practice that aims at unifying software development (Dev) and
Software operation (Ops). The main characteristic of the DevOps movement is to
strongly advocate automation and monitoring at all steps of software
construction, from integration, testing, releasing to deployment and
infrastructure management.

The above definition describes the kops team's vision for developing kops.
Repeatable, tested automation is the core of the philosophy of DevOps.  Through
code the kops' team does the hard things to make it easy for the kops users.

kops is tested, tested and tested again.  Every time a PR is pushed in the main
Kuberentes repository, kops builds a cluster in AWS.  The end to end kubernetes
tests are then run on that cluster.  The kops team is working on getting GCE e2e
tests up and running soon.

A key component of DevOps culture is repeatable processes, and cluster
configurations can get complicated.  One of the problems using CLI tools, in my
opinion, is flag hell.  `kops create cluster` has close to 40 different CLI flags.
Enter `kops create -f mycluster.yaml`, to save the day.  kops just as Kubernetes
can be driven by YAML or JSON [manifests](https://github.com/kubernetes/kops/blob/master/docs/manifests_and_customizing_via_api.md) that can be checked into source control.
Coming in kops 1.8.0 the kops team is introducing various templating features as
well to help build the manifests.

## kops Features

### Supported Clouds

- AWS
- GCE in kops 1.8.0
- Alpha support for vSphere
- Digital Ocean and Bare metal is under development

### Features

- Deploys Highly Available (HA) Kubernetes Masters
- Ability to generate configuration files for Terraform and AWS CloudFormation
- Supports custom Kubernetes add-ons
- Command line autocompletion
- Manifest Based API Configuration
- Rolling updates and upgrades that are on par with commercial products
- Validate that Kubernetes is up and running
- Exporting kubeconfig for installed clusters
- CLI options for edit, create, delete, update, upgrade, rolling-update,
  validate, and more
- Full support for RBAC

## Tutorials

I am not going to cover how to install clusters, in this post, because the kops
the project has multiple different tutorials:

- [Amazon Web Services](https://github.com/kubernetes/kops/blob/master/docs/aws.md)
- [Google Compute Engine](https://github.com/kubernetes/kops/blob/master/docs/tutorial/gce.md)

Google Compute Engine is still under development, and with the 1.8.0 release
will be stable.  As of the time of writing this post 1.8.0 is not
[released](https://github.com/kubernetes/kops/releases) yet. Either compile
master or use the 1.8.0 alpha.
