---
title: Google Cloud Hosting
description: How I am hosting this blog in Google Cloud
header: Setting Up Google Cloud To Serve This Blog
date:   2017-10-14 16:52:54 MDT
categories: blog
tags:
  - howto
  - google cloud
---

[chrislovecnm.com](https://chrislovecnm.com) needed an internet home. Using
Jekyll creates a static HTML/JS/CSS site, which makes hosting from an Object
Store simple.

## Choices

There are a lot of companies out there that offer object stores.  I am daily
using AWS Cloud and GCE developing kops and Kubernetes.  I could have chosen
something else, but I decided to stay with the major players.  Frankly, I could
not decide.  My wife made the final call.  The conversation was: "Hon should we
use AWS or Google?".  Her answer was Google.

## Sign-up

Here is the link [free trial](https://console.cloud.google.com/freetrial) for
the free trial.  At the time I am writing this post, the free period is good for
$300 over a year. From what I could figure out both I could not use AWS Free
Tier since they charge for DNS.

## Storage

S3 is a very well known Object Storage Amazon Cloud service. Google's option is
[Google Storage](https://cloud.google.com/storage/).

Setting up a static site is trivial and
[documented](https://cloud.google.com/storage/docs/hosting-static-website).  But
that is fine if you are ok with two things.

1. Using HTTP
1. Using subdomain like www.chrislovecnm.com

My requirements

1. Use HTTPS, because Google gives better SEO with HTTPS
1. www subdomain is so 2000's.  Just use chrislovecnm.com

### Uploading Content

This blog's Travis CI is not set up yet to copy content over, and I am kind of
on the fence if I am going to do that.  For now:

{% highlight bash %}
gsutil -m rsync -a public-read -r _site/ gs://$YOURSITEBUCKET/
{% endhighlight %}

The Google storage UX does not have an option to make all files in a folder accessible publicly, but you can set the default permission to the public.  Run the following command.

{% highlight bash %}
gsutil defacl ch -u AllUsers:R gs://$YOURSITEBUCKET
{% endhighlight %}

Replace `$YOURSITEBUCKET` with your bucket name.

## Load Balancing

Using a Load Balancer allows you to have HTTP, HTTPS, and an IP address.  With
an IP address, you can do anything you want with DNS.

Google has clear
[instructions](https://cloud.google.com/compute/docs/load-balancing/http/adding-a-backend-bucket-to-content-based-load-balancing)
for setting up a Load Balancer using serving content from a bucket in Cloud Storage.

I have both HTTP and HTTPS setup, which required me to create a static IP. SSL
certificates are always a PAIN.  [LetsEncrypt](https://letsencrypt.org/) is
trying to change that, and with some trial and error finally got it configured.

## SSL Certificates

My road to SSL was a bit of a wandering path.  In order, to get a cert from
LetsEncrypt you have to use there API. Lots of options exist, so I tried two
different web based online tools.  I pulled my hair out for about 6 hours.  I am not
exaggerating.

Then I finally found this gem: [Google Cloud HTTPs load balancing with Letsencrypt certificate](https://rubyinrails.com/2017/09/18/google-cloud-https-load-balancing-with-letsencrypt-certificate/).

### TLDR;

- Install [certbot](https://github.com/certbot/certbot), and there is a brew package.
- Run `sudo certbot certonly --manual -d yoursite.com,www.yoursite.com`
- Follow the instructions and upload the web pages to your site
- Configure the load balancer with the cert.pem, fullchain.pem, and privkey.pem

certbot CLI options and the name of the files it creates have changed since the
post on ruby in rais, was initially written, so it took a bit of playing. But
less than six hours.

With the `gcloud` CLI you can follow these
[instructions](https://cloud.google.com/compute/docs/load-balancing/http/ssl-certificates#createresource),
but I used clicked on the buttons in the cloud console.  The general
instructions for setting up the load balancer with HTTPS are
[here](https://cloud.google.com/compute/docs/load-balancing/tcp-ssl/#configure_load_balancer),
but the UX wizards at Google have made it simple.

## DNS

DNS is dead simple with Google Cloud, but not as many bells and whistles as such
services as AWS Cloud Route53 or CloudFlare.  But simple is good.  I switched my
domain's NS servers with my registrar and set the static IP address assigned to
my load balancer as a DNS "A" record.

## Improvements

- Need to move another domain over to Google Cloud, and figure out how to use the same load balancer.
- Google Analytics is wonky. Referral and page names are messed up, but I would sooner write more posts, than worry about the traffic numbers.
