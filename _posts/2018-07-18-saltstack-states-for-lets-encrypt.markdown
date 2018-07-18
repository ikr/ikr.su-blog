---
layout: post
title: "SaltStack states for Let's Encrypt certificates"
date: 2018-07-18 11:00:00
tags:
  - saltstack
  - ssl
  - tls
  - letsencrypt
  - nginx
  - debian
  - linux
---

Recently I've configured [Let's Encrypt certificates](https://letsencrypt.org/) for a staging HTTPS
server for one of our clients. That was amazingly easy to do. I remember checking the ACME protocol
client tools in the very beginning of the Let's Encrypt initiative. The tools were, of course, very
immature: hard to install and configure, and they only worked in interactive mode. It's all in the
past now.

Basically, I took [this great blog
post](https://techtalk.blog/simple-and-free-ssl-certificates-using-letsencrypt-and-nginx-530f03aee07)
and codified it in [SaltStack
states.](https://docs.saltstack.com/en/latest/topics/tutorials/starting_states.html) SaltStack is an
automatic configuration management tool similar to
[Puppet](https://puppet.com/solutions/configuration-management) and
[Ansible](https://www.ansible.com/use-cases/configuration-management). It is quite simple and
declarative, yet it scales up to larger infrastructure. Our sysadmins love it.

Below is the annotated YAML of my solution.

{% highlight yaml %}
backports:
  pkgrepo.managed:
    - name: deb http://ftp.de.debian.org/debian jessie-backports main contrib non-free
    - file: /etc/apt/sources.list.d/backports.list

dehydrated:
  pkg.installed:
    - fromrepo: jessie-backports
    - require:
      - pkgrepo: backports
{% endhighlight %}

That adds the jessie-backports Debian package repository, and installs the
[Dehydrated](https://dehydrated.io/) ACME client from it. Starting with Debian Stretch (v9) it's
[not necessary to add the backports
repo.](https://packages.debian.org/search?keywords=dehydrated&searchon=names&suite=all&section=all)
Then, the section above can be reduced to just this:

{% highlight yaml %}
dehydrated:
  pkg.installed
{% endhighlight %}
