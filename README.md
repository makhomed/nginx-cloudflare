# nginx-cloudflare (version 1.2.4)

Set client real IP for nginx behind Cloudflare.

Since Cloudflare acts as a reverse proxy, all connections come from one of Cloudflare's IP addresses.
Cloudflare includes the originating IP address in the [X-Forwarded-For](https://developers.cloudflare.com/fundamentals/reference/http-headers/#x-forwarded-for) and
[CF-Connecting-IP](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip) headers.

This script, enabled in the cron, allow automatically maintaining an up to date
[list of Cloudflare ip addresses](https://www.cloudflare.com/ips/)
and reloading nginx if these addresses changed after update.

File `/etc/nginx/include/cloudflare_set_realip_from.conf` contains generated [set_real_ip_from](https://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from) directives for IP addresses from Cloudflare networks.

File `/etc/nginx/include/cloudflare_not_deny_access.conf` contains generated fragment for [nginx geo module](https://nginx.org/en/docs/http/ngx_http_geo_module.html),
allow blocking all non-Cloudflare IP addresses, using `if` directive from [nginx rewrite module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html).

## Installation
> [!IMPORTANT]
> Python 3.9+ and [Jinja2](https://jinja.palletsprojects.com/), [requests](https://requests.readthedocs.io/), [invoke](https://www.pyinvoke.org/) modules required
```
dnf -y install python3 python3-pip ; \
python3 -m pip install --no-input --upgrade-strategy eager --upgrade Jinja2 requests invoke ; \
cd /opt ; git clone https://github.com/makhomed/nginx-cloudflare.git
```

## Upgrade
```
python3 -m pip install --no-input --upgrade-strategy eager --upgrade Jinja2 requests invoke ; \
cd /opt/nginx-cloudflare ; git pull
```

## Usage
```
/opt/nginx-cloudflare/nginx-cloudflare
```

## Automation via cron

Configure cron job, for example, in file `/etc/cron.d/nginx-cloudflare`:

```
RANDOM_DELAY=60

0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare
```

## nginx configuration in http context
```
    include /etc/nginx/include/cloudflare_set_realip_from.conf;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    geo $realip_remote_addr $request_not_from_cloudflare {
        include /etc/nginx/include/cloudflare_not_deny_access.conf;
    }
```

## nginx configuration in server context
```
    if ($request_not_from_cloudflare) { return 444; }
```

