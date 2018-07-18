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

The `salt://web-server/conf/example.com` nginx configuration file:

{% highlight nginx %}
server {
    listen 80;
    server_name example.com;

    include /etc/nginx/conf.d/*.http.location.inc;

    location / {
        return 301 https://example.com$request_uri;
    }
}

server {
    listen 443 default_server ssl;
    server_name example.com;

    ssl on;
    include /etc/nginx/conf.d/ssl.inc;
    include /etc/nginx/conf.d/*.location.inc;
    …
}
{% endhighlight %}

Which is then referenced from the `salt://web-server/init.sls`:

{% highlight yaml %}
/etc/nginx/sites-enabled/example.com:
  file.managed:
    - source: salt://web-server/conf/example.com
    - require:
      - pkg: nginx
    - watch_in:
      - service: nginx

/etc/nginx/conf.d/acme-challenge.http.location.inc:
  file.managed:
    - source: salt://web-server/conf/acme-challenge.http.location.inc
    - require:
      - pkg: nginx
      - pkg: dehydrated
    - watch_in:
      - service: nginx
{% endhighlight %}

The `salt://web-server/conf/acme-challenge.http.location.inc` file is nothing but:

{% highlight nginx %}
location ^~ /.well-known/acme-challenge {
    auth_basic "off";
    alias /var/lib/dehydrated/acme-challenges;
}
{% endhighlight %}

Now the tricky bit. The `/etc/nginx/conf.d/ssl.inc` referenced in the
`salt://web-server/conf/example.com` will in the end contain the path to the certificate file, and
the path to the secret server key file. We don't have those initially, before the ACME challenge
takes place. Thus, need some “transitional” certificate and key to make nginx start and serve the
ACME challenge Web location over HTTP.

{% highlight yaml %}
ssl_inc_transitional:
  cmd.run:
    - name: >
        echo 'ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;' > /etc/nginx/conf.d/ssl.inc &&
        echo 'ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;' >> /etc/nginx/conf.d/ssl.inc
    - unless: test -f /etc/nginx/conf.d/ssl.inc
    - require:
      - pkg: nginx
    - require_in:
      - service: nginx

/etc/dehydrated/domains.txt:
  file.managed:
    - contents:
      - example.com
    - require:
      - pkg: dehydrated

initial_lets_encrypt_cert:
  cmd.run:
    - name: /usr/bin/dehydrated --cron
    - unless: test -d /var/lib/dehydrated/certs/example.com
    - require:
      - file: /etc/nginx/conf.d/acme-challenge.http.location.inc
      - file: /etc/dehydrated/domains.txt

ssl_inc:
  cmd.run:
    - name: >
        /bin/systemctl stop nginx &&
        echo 'ssl_certificate /var/lib/dehydrated/certs/example.com/fullchain.pem;' > /etc/nginx/conf.d/ssl.inc &&
        echo 'ssl_certificate_key /var/lib/dehydrated/certs/example.com/privkey.pem;' >> /etc/nginx/conf.d/ssl.inc &&
        /bin/systemctl start nginx
    - onlyif: test -f /etc/nginx/conf.d/ssl.inc && cat /etc/nginx/conf.d/ssl.inc | grep -F ssl-cert-snakeoil
    - require:
      - cmd: initial_lets_encrypt_cert
{% endhighlight %}

The final part is the certificate renewal cron job:

{% highlight yaml %}
root_email_for_cron:
  cron.env_present:
    - user: root
    - name: MAILTO
    - value: webmaster@example.com

lets_encrypt_cert_update:
  cron.present:
    - name: chronic /usr/bin/dehydrated --cron && systemctl reload nginx
    - identifier: lets_encrypt_cert_update
    - user: root
    - dayweek: 0
    - hour: 1
    - minute: 2
{% endhighlight %}
