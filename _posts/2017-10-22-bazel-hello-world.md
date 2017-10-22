---
title: Bazel Hello World
description: Setting up Bazel with golang
header: Bazel Go Hello World
date:   2017-10-22 12:27:01 MDT
categories: golang bazel
tags:
  - bazel
  - golang
---

[Bazel](https://bazel.build) is seriously amazing.  The learning curve is a bit steep, but once @justinsb
and I got it running in kops.  Holy Cow Batman.  The primary kubernetes/kubernetes
repositories has had bazel enabled for months, but I had run into problems with
building with bazel on my MacBook Pro.

![Bazel Logo](/img/bazel-icon.svg){:class="img-responsive"}

See this [link](https://blog.bazel.build/2017/07/05/new-logo-and-homepage.html) for information on there logo.

## Why Bazel

1. Speed Speed and more Speed.  Because of caching, compilation and unit test
  speed is rediculus.
1. One tool to rule them all.  Bazel supports Go, Java, C++, Android, iOS, on macOS
  Linux, Windows.
1. Extensabilty - Add plugins and call external tools.
1. The component scales.  It handles codebases of any size, and integrates into CI.

Blaze is Goolge's closed source build tool, and is the predicesor to bazel.  So I
know that the above claims are true.

## Using Bazel with Go

### Pre-flight Checks

1. Install bazel - Instructions are [here](https://docs.bazel.build/versions/master/install.html).
2. It is my understanding that you do not even need to install go, but it is helpful otherwise. [Here](https://golang.org/doc/install) are install instructions for go.

### Helper Script

Create your project with your source control tool of choice.  Inside that
project run the following bash script for this tutorial.

Provide the go path for the project.  For example: `create-bazel-workspace github.com/myuser/myproject`.

{% highlight bash %}
create-bazel-workspace() {
PREFIX=$1
cat &gt; WORKSPACE &lg;&lg;- EOM
git_repository(
    name = "io_bazel_rules_go",
    remote = "https://github.com/bazelbuild/rules_go.git",
    tag = "0.6.0",
)

load("@io_bazel_rules_go//go:def.bzl", "go_rules_dependencies", "go_register_toolchains", "go_repository")
go_rules_dependencies()
go_register_toolchains()

go_repository(
    name = "com_github_golang_glog",
    commit = "23def4e6c14b4da8ac2ed8007337bc5eb5007998",
    importpath = "github.com/golang/glog",
)

go_repository(
    name = "com_github_spf13_cobra",
    commit = "7b1b6e8dc027253d45fc029bc269d1c019f83a34",
    importpath = "github.com/spf13/cobra",
)

go_repository(
    name = "com_github_spf13_pflag",
    commit = "f1d95a35e132e8a1868023a08932b14f0b8b8fcb",
    importpath = "github.com/spf13/pflag",
)
EOM
cat &gt; BUILD &lg;&lg;- EOM
load("@io_bazel_rules_go//go:def.bzl", "go_prefix", "gazelle")

go_prefix("${PREFIX}")

# bazel rule definition
gazelle(
  prefix = "${PREFIX}",
  name = "gazelle",
  command = "fix",
)
EOM
}
{% endhighlight %}

The above script will seed a project to use [cobra](https://github.com/spf13/cobra).
I have provided an example project located [here](https://github.com/chrislovecnm/go-bazel-hello-world) on Github.

#### The Files

The above script creates two files.  The "WORKSPACE" file is maniditory for every
bazel project. This file typically contains references to external build rules and
project dependencies.

From my example project:

{% highlight bazel %}
{% include github.com/chrislovecnm/go-bazel-hello-world/WORKSPACE %}
{% endhighlight %}

The syntax quite simliar to groovy, and you can see the dependencies being added
to the Workspace.  This is one of the areas that needs some automation, and hopefully
be adderess by improving this [issue](https://github.com/bazelbuild/rules_go/issues/389).

The next set of files are called "BUILD".

From bazel [documentation](https://docs.bazel.build/versions/master/build-ref.html#BUILD_files):

> By definition, every package contains a BUILD file, which is a short program written in the Build Language. Most BUILD files appear to be little more than a series of declarations of build rules; indeed, the declarative style is strongly encouraged when writing BUILD files.

Here is the BUILD file in the root directory of the example project.

{% highlight bazel %}
{% include github.com/chrislovecnm/go-bazel-hello-world/BUILD %}
{% endhighlight %}

The BUILD and Build.bazel files are the continual work in bazel.  Add a new
import in go, and you need to update the BUILD file.  Thankfully the bazel
authors have added gazelle.

### Gazelle Build File Generator

Once you have the project initiallized, you can execute `bazel run //:gazelle`.
This will execute `gazelle`, because the gazelle rule is defined in the above
BUILD file.  This will create various BUILD.bazel files in your project.

More documentation about [Gazelle](https://github.com/bazelbuild/rules_go/blob/master/go/tools/gazelle/README.rst).

### Defining Your Binary File(s)

One thing that gazelle did not do intially was define a binary.  I added
the `go_binary` and `go_library` rules by hand, after gazelle generated the files.

{% highlight bazel %}
{% include github.com/chrislovecnm/go-bazel-hello-world/cmd/BUILD.bazel %}
{% endhighlight %}


### Common Bazel Commands

The example Makefile contains the usual suspects for bazel commands.

{% highlight make %}
{% include github.com/chrislovecnm/go-bazel-hello-world/Makefile %}
{% endhighlight %}

## Next Steps

Bazel is powerful. You can do cool things like running external targets, and
building containers without docker, a heck of alot faster than docker does it!

Cross compiling with bazel is complicated, and the support for making Linux
binaries on OSX is limited. One solution is to use a container such as [planter](https://github.com/kubernetes/test-infra/tree/master/planter).

I am planning on futher posts about using go-bin-data and containers with bazel
once we have those fixed in kops.

## TLDR;

1. Install bazel
1. Run the script or create the Workspace and BUILD files
1. Execute `bazel run //:gazelle`
1. Define your binaries

## Thanks

First off thanks to the bazel team, for such an amazing tool. Thanks to @justinsb
for his [kopeio/build](https://github.com/kopeio/build) project.  Was able to
work through issues for my example, using his project as a base. @ixby, @bentheelder
and others help on the #bazel Kubernetes slack channel has been invaliable.
