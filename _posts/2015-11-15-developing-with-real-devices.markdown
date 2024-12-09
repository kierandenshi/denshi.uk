---
layout: post
title:  "Developing with real devices"
date:   2015-11-15 18:05:21 +0000
categories: dns pdnsd hosts ios
published: true
---
When developing locally or on a server without a domain name or accessible IP address and you want to test with real mobile devices and not a simulator, you can use your mac as a dns server and use then your local `/etc/hosts` file to add routing entries.

```sh
brew install pdnsd
```

Now chown the cache directory

```sh
sudo chown -R nobody /usr/local/var/cache/pdnsd
```

And create a conf file

```sh
nano /usr/local/etc/pdnsd.conf
```

And create a basic configuration, example:

```conf
global {
    perm_cache=2048;
    cache_dir="/usr/local/var/cache/pdnsd";
    max_ttl=604800;
    run_as="nobody";
    paranoid=on;
    server_port=53;
    server_ip="0.0.0.0";
    run_as = nobody;
}

server {
    label=OpenDNS;
    ip=208.67.222.222;
    ip=208.67.220.220;
    timeout=30;
    interval=30;
    uptest=ping;
    ping_timeout=50;
    purge_cache=off;
}

source {
    owner=localhost;
    serve_aliases=on;
    file="/etc/hosts";
}
```

Now chown the conf file to root

```sh
sudo chown root /usr/local/etc/pdnsd.conf
```

Start it up with

```sh
sudo pdnsd
```

Now set the dns on your device to use your mac's ip address. Your phone and tablet will now be able to use your mac for dns and use any entries you make in `/etc/hosts`.


[1]: https://brew.sh/
