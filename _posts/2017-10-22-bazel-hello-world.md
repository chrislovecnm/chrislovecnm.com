---
title: Bazel Golang Hello World
description: Setting up Bazel with the Go Programming Language
header: Bazel Golang Hello World
date:   2017-10-22 12:27:01 MDT
categories: golang bazel
tags:
  - bazel
  - golang
  - go
image: /img/bazel-icon.svg
author:
  twitter: chrislovecnm
---

> Warning: only use if you have a severe need for speed and want to be more
productive.

[Bazel](https://bazel.build) is seriously amazing.  The primary
Kubernetes/Kubernetes repository has had Bazel enabled for months, but I had run
into problems as far as building with Bazel on my MacBook Pro. The learning curve to
use bazel is a bit steep, but once @justinsb and I got it running in kops: Holy
Cow Batman!  Kops tests and builds are so much faster. When cached, build times
are sub-second which took 6 min.  Those numbers are no joke.

![Bazel Logo](/img/bazel-icon.svg){:class="img-responsive"}

See this [link](https://blog.bazel.build/2017/07/05/new-logo-and-homepage.html)
for information on their logo.

## What is Bazel

On February 25th, 2015, a Googler pushed the first comment to the Bazel project,
and Google officially Open Sourced the project.  Blaze is Google's closed source
build tool and is the predecessor to Bazel.

Bazel is a build tool that replaces using Makefiles or other tools for building
go.  Under the hood, it uses `go build`; but it is not your average build tool. 
Just as `make` has many different options, Bazel provides many exciting features
including dependency management, and templating with external tools, and the
capability to build containers without docker.

## Why Bazel

1. Speed, speed and more speed, Ferrari-like performance. Because of caching,
compilation and unit test speed is ridiculous.
1. One tool to rule them all. Bazel supports Go, Java, C++, Android, iOS, on
 OSX, Linux, and Windows.
1. Extensibility. Add plugins and call external tools.
1. The tool scales. It handles codebases of any size and integrates into CI.
  The Kubernetes/Kubernetes repository is a huge repo with multiple containers
  and binaries.
1. Build vendoring for go.  We were not able to fully use it in kops because
 of some challenges, but for other projects it works great. Watch out dep, cause
 you have a serious competitor.

## Using Bazel with Go

### Pre-flight Checks

1. Install Bazel - Instructions are [here](https://docs.bazel.build/versions/master/install.html).
2. It is my understanding that you do not even need to install Golang, but it is
 helpful otherwise. [Here](https://golang.org/doc/install) are the install
 instructions for Golang.

### Helper Script

Create your project with your source control tool of choice.  Inside your
project, run the bash script provided
[here](https://github.com/chrislovecnm/go-bazel-hello-world/blob/master/hack/create-bazel-workspace.sh).

Provide the go path for the project.  For example: `create-bazel-workspace
github.com/myuser/myproject`

Here is the script:

{% highlight bash %}
{% include github.com/chrislovecnm/go-bazel-hello-world/hack/create-bazel-workspace.sh %}
{% endhighlight %}

The above script will seed a project to use
[cobra](https://github.com/spf13/cobra). I have provided an example project
located [here](https://github.com/chrislovecnm/go-bazel-hello-world) on Github.

#### The Files

The above script creates two files.  The "WORKSPACE" file is mandatory for every
Bazel project. This file typically contains references to external build rules
and project dependencies.

From my example project:

{% highlight bazel %}
{% include github.com/chrislovecnm/go-bazel-hello-world/WORKSPACE %}
{% endhighlight %}

The syntax quite similar to Groovy, and you can see the go dependencies being
added to the Workspace.  Continually adding `go_repository` rules is one of the
areas that needs some automation, and hopefully be addressed with work on this
[issue](https://github.com/bazelbuild/rules_go/issues/389).

The next set of files are called BUILD, or BUILD.bazel.

From Bazel
[documentation](https://docs.bazel.build/versions/master/build-ref.html#BUILD_files):

> By definition, every package contains a BUILD file, which is a short program
written in the Build Language. Most BUILD files appear to be little more than a
series of declarations of build rules; indeed, the declarative style is strongly
encouraged when writing BUILD files.

Here is the BUILD file in the root directory of the example project.

{% highlight bazel %}
{% include github.com/chrislovecnm/go-bazel-hello-world/BUILD %}
{% endhighlight %}

The BUILD and BUILD.bazel files are the continual work in Bazel.  Add a new
import in go, and you need to update the BUILD file. Add a new go file or test,
and yep, update BUILD.bazel.  Thankfully the Bazel authors have added Gazelle.

### Gazelle Build File Generator

Once you have the project initialized, you can execute `bazel run //:gazelle`.
This command will execute `gazelle` because the Gazelle rule is defined in the
above BUILD file.  Running Gazelle will create various BUILD.bazel files in your
project.

More documentation about
[Gazelle](https://github.com/bazelbuild/rules_go/blob/master/go/tools/gazelle/README.rst).

### Defining Your Binary File(s)

One thing that Gazelle did not do initially was define a rule to build a binary.
I added the `go_binary` and `go_library` rules by hand, after Gazelle, generated
the files.

{% highlight bazel %}
{% include github.com/chrislovecnm/go-bazel-hello-world/cmd/BUILD.bazel %}
{% endhighlight %}


### Common Bazel Commands

The example Makefile contains the usual suspects for Bazel commands.  The usual
workflows for development: build, test, and Gazelle for Bazel.

{% highlight make %}
{% include github.com/chrislovecnm/go-bazel-hello-world/Makefile %}
{% endhighlight %}

## Next Steps

Bazel is powerful. You can do cool things like running external targets, and
building containers without docker, a heck of a lot faster than docker does it!

Cross-compiling with Bazel is complicated, and the support for making Linux
binaries on OSX is limited. One solution is to use a container such as
[planter](https://github.com/kubernetes/test-infra/tree/master/planter).

I am planning on further posts about using go-bindata and containers with Bazel
once we have those fixed in kops.

We need to integrate Bazel into kops testing with Travis.  One note, use caching
inside your CI tool.  Without caching you loose ALL of Bazel's performance.
Frankly you may as well use `go build`.

## TLDR;

1. Install Bazel
1. Run the script or create the Workspace and BUILD files
1. Execute `bazel run //:gazelle`
1. Define your binaries with Bazel rules
1. Enjoy the serious performance

## Thanks

First off thanks to the Bazel team, for such an amazing tool. Thanks to
@justinsb for his [kopeio/build](https://github.com/kopeio/build) project.  I
was able to work through issues for my example, using the project as a base.

@ixby, @bentheelder, and others help on the #bazel Kubernetes slack channel have
been invaluable.

Thanks to @jroberts235 for recommending including a basic overview of Bazel.
