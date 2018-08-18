---
title: Jekyll Building This Blog
description: How I used Jekyll to build this blog
header: Building This Blog with Jekyll
date:   2017-10-14 14:52:14 MDT
categories: jekyll building blog
tags: jekyll building blog
---

[Jekyll](https://jekyllrb.com/) has crossed my path in the past, and I decided
to start blogging again. Jekyll was my obvious choice.  Wordpress would be my
other option, but that requires a web server.  I would rather just have a static
site for now.

## Tools I Am Using

1. [Jekyll](https://jekyllrb.com/) for blog generation
1. Jekyll theme
1. GitHub for source control
1. Travis for CI
1. html-proofer for checking links
1. atom or vi for editing
1. Google analytics for tracking
1. Grammarly to help with my spelling

## Jekyll

From [here](https://jekyllrb.com/):

> Transform your plain text into static websites and blogs.

It is called simple on their site, but there is a learning curve.  If you are a
gearhead developer that like CLI, the Jekyll is amazing.  If you are a
marketing department, use Wordpress.

The `_config.yml` configures Jekyll, and some tweaks that took a bit to figure
out:

{% highlight yaml %}
# setup the link structure generated
# this requires categories and title set in your pages
permalink: /:categories/:title/
# exclude files from being published to your site
exclude: ['README.md', 'Gemfile.lock', 'Gemfile', 'vendor', 'Rakefile']
{% endhighlight %}

Here is a link to my current
[configuration](https://github.com/chrislovecnm/chrislovecnm.com/blob/master/_config.yml).

## Jekyll Theme

[Jekyllthemes.org](http://jekyllthemes.org/) has a ton of free themes. My wife
did not like the colors for a solarized theme, but did like
[monochrome](https://github.com/dyutibarma/monochrome).  Clean, simple layout. I
have some issues with CSS for code line numbers which I am not able to work out
yet, and I made some tweaks for tables.

## Github

Do I have to write much about this? But using source control is a way of
life for me.  Using source control with Jekyll is trivial.

## Travis for CI

Travis, CircleCI, and Jenkins are probably the chief tools in the ecosystem.
Kops, and a lot of the other projects I am a part of run Travis.  Hence why I
chose Travis. I started with the instructions
[here](http://jekyllrb.com/docs/continuous-integration/travis-ci/), but I
modified my .travis.yml file.

{% highlight yaml %}
language: ruby
rvm:
- 2.3.3
# Assume bundler is being used, therefore
# the `install` step will run `bundle install` by default.
script:
- bundle exec jekyll build
- bundle exec htmlproofer ./_site --assume-extension --check-html
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer
# route your build to the container-based infrastructure for a faster build
sudo: false
{% endhighlight %}

I do not have the automatic deployment of my blog yet.  Travis can push the blog
to an object store, but I have not worked it out fully yet.

# html-proofer

This HTML tool is super nice!  Check your links, and HTML validation.  A
spelling and grammar tool would be awesome.

You can run it locally.  I got a weird error that the binary was missing when I
tried to run it locally.  Besides, why would I not use a container to run it?

{% highlight bash %}
docker run -v `pwd`/_site/:/_site 18fgsa/html-proofer /_site \
 --assume-extension --check-html
{% endhighlight %}

## Editor

Not going to elaborate, but with Jekyll, you are editing markdown and YAML
files.  Pick your favorite editor and rock it!  My choice is vi or atom.

## Google Analytics

I will go into more details when I blog about my hosting solution, but I have
used GA for years.

## Grammarly

[Grammarly](https://app.grammarly.com/) helps me a TON. I am good at writing,
but spelling and past tense are enemies!  Grammarly is free for spelling and
other checks, but checking for "past tense" requires a subscription.  I would
like to figure out how to integrate Grammarly with atom or vi.  Just googled
online, and NADA.

## Next Steps

[Chrislovecnm.com](https://chrislovecnm.com) needed an internet home.  I will
cover that in another post.
