# NGINX

This document describes how Curiefense can be integrated into an existing NGINX-based reverse proxy.

{% hint style="info" %}
This guide describes a basic integration, and it cannot cover the wide variety of possible use cases and configurations. \
\
For specific questions about this, or other Curiefense-related topics, feel free to join our Slack at [https://join.slack.com/t/curiefense/shared_invite/zt-nc8lyrjo-JJoY2mwrqNOfkmoA6ycTHg](https://join.slack.com/t/curiefense/shared_invite/zt-nc8lyrjo-JJoY2mwrqNOfkmoA6ycTHg).
{% endhint %}

## Scope

This page describes the installation of the Curiefense filtering component for an environment where NGINX is running in a container. 

The other components of Curiefense will need to be installed separately, according to the specific instructions for each situation (e.g., [Docker](docker-compose.md) and [Istio](istio-via-helm.md)). This can be done either before or after completing the instructions below.

## Dependencies

If OpenResty is not installed yet, please follow the [instructions on the OpenResty website](https://openresty.org/en/installation.html).

You will also need the [Hyperscan library](https://github.com/intel/hyperscan), version 4 or 5. For example, on Ubuntu 20.04:

```bash
sudo apt install libhyperscan5
```

## Curiefense installation

```bash
cd /tmp
git clone https://github.com/curiefense/curiefense
cp -r /tmp/curiefense/curiefense/curieproxy/lua /
cp /tmp/curiefense/curiefense/curieproxy/lua/shared-objects/grasshopper.so /usr/local/openresty/luajit/lib/lua/5.1/
```

Next, build the Curiefense shared object. This needs to be done on a Linux system that runs the same major `libc` and `libhyperscan` versions as your NGINX server. 

* On the build machine, first [install the Rust compiler](https://www.rust-lang.org/tools/install).
*   Then run the following:

    ```
    cd /tmp/curiefense/curiefense/curieproxy/rust
    cargo build --release
    mv target/release/libcuriefense_lua.so target/release/curiefense.so
    ```
* Move the new `curiefense.so` file on the build machine to this location on the proxy machine: `/usr/local/openresty/luajit/lib/lua/5.1/curiefense.so`

## Configuration setup

```
mkdir -p /config/current
cp -r /tmp/curiefense/curiefense/images/confserver/bootstrap/confdb-initial-data/master/config/ /config/current/
```

## OpenResty configuration

In the http block of the configuration, the following directives must be set:

```
# set the package path for the Curiefense logic
lua_package_path '/lua/?.lua;;';

# access log is now completely handled by Curiefense, stored in the $request_map variable
log_format curiefenselog escape=none '$request_map';

# Curiefense specific variables.
# Make sure request_map exists
map $status $request_map { default '-'; }
```

For each server block that must be protected with Curiefense:

```
server {
    # original configuration directives
    (...)

    
    # Logging: choose one option.
 
    # -- If curielogger is used:
    access_log syslog:server=unix:/dev/log curiefenselog;
    
    # -- If curielogger is not used:
    access_log logs/access.log curiefenselog;


    location / {
        # add these in each location block:
        access_by_lua_block {
            local session = require "lua.session_nginx"
            session.inspect(ngx)
        }
        log_by_lua_block {
            local session = require "lua.session_nginx"
            session.log(ngx)
        }

        # original location directives
        (...)
    }
}
```

## Testing the installation

The default configuration does not block any requests, so the following steps should be performed to ensure proper integration:

* Traffic should be served as usual.
* No errors appear in the error logs.
* JSON data appears in the access logs.
