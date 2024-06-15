# nginx-cloudflare (version 1.2.3)

Set client real IP for nginx behind Cloudflare.

Since Cloudflare acts as a reverse proxy, all connections come from one of Cloudflare's IP addresses.
Cloudflare includes the originating IP address in the
[`CF-Connecting-IP`](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip)
header.

This script, enabled in the cron, allow automatically maintaining an up to date
[list of Cloudflare ip addresses](https://www.cloudflare.com/ips/)
and reloading nginx if these addresses changed after update.

First block of generated files, `/etc/nginx/include/cloudflare.conf`,
`/etc/nginx/include/cloudflare_ipv4.conf`, `/etc/nginx/include/cloudflare_ipv6.conf`,
contains settings for [nginx realip module](https://nginx.org/en/docs/http/ngx_http_realip_module.html),
allowing nginx to restore real client IP from the
[`CF-Connecting-IP`](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip)
header.

Second block of generated files, `/etc/nginx/include/from_cloudflare.conf`,
`/etc/nginx/include/from_cloudflare_ipv4.conf`, `/etc/nginx/include/from_cloudflare_ipv6.conf`,
contains settings for [nginx realip module](https://nginx.org/en/docs/http/ngx_http_realip_module.html),
but only [`set_real_ip_from`](https://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from) directives,
allowing to manually configure [`real_ip_header`](https://nginx.org/en/docs/http/ngx_http_realip_module.html#real_ip_header) directive
to use [`X-Forwarded-For`](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#x-forwarded-for) header
from Cloudflare or [`True-Client-IP`](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#true-client-ip-enterprise-plan-only)
header from Cloudflare or [`CF-Connecting-IP`](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip) header
from Cloudflare, or any other future [HTTP request headers, added by Cloudflare to proxied request from
client](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/).

Third block of generated files, `/etc/nginx/include/not_cloudflare.conf`,
`/etc/nginx/include/not_cloudflare_ipv4.conf`, `/etc/nginx/include/not_cloudflare_ipv6.conf`,
contains settings for [nginx geo module](https://nginx.org/en/docs/http/ngx_http_geo_module.html),
allow blocking all non-cloudflare IPs in server context, blocking direct access to site,
if anyone will try to bypassing Cloudflare. Such blocking at nginx level may be required,
if [`nftables-tool`](https://github.com/makhomed/nftables-tool/) not used,
and system nftables permits simultaneously both types of connections - for some sites -
direct connections from clients, and for some sites - only proxied connections via IP addresses from Cloudflare networks.
In this case - `not_cloudflare.conf` files can help to permit only requests from Cloudflare IPs for some sites with minimal overhead.

## Installation
> [!IMPORTANT]
> Python 3.8+ and [Jinja2](https://jinja.palletsprojects.com/en/3.1.x/),
> [requests](https://requests.readthedocs.io/),
> [invoke](https://www.pyinvoke.org/) modules required
```

dnf -y install python3 python3-pip python-unversioned-command ; \
python -m pip install --no-input --upgrade-strategy eager --upgrade Jinja2 requests invoke ; \
cd /opt ; git clone https://github.com/makhomed/nginx-cloudflare.git
```

## Upgrade
```
python -m pip install --no-input --upgrade-strategy eager --upgrade Jinja2 requests invoke ; \
cd /opt/nginx-cloudflare ; git pull
```

## Usage
```
/opt/nginx-cloudflare/nginx-cloudflare
```
or
```
/opt/nginx-cloudflare/nginx-cloudflare 172.21.0.0/16 2001:DB8:11:22::/64 2001:DB8:99:77::/64
```

## Automation via cron

Configure cron job, for example, in file `/etc/cron.d/nginx-cloudflare`:

```
RANDOM_DELAY=360

0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare
```
or
```
RANDOM_DELAY=360

0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare 172.21.0.0/16 2001:DB8:1:2::/64 2001:DB8:8:9::/64
```

## nginx configuration in http context
```
    include /etc/nginx/include/cloudflare.conf;

    geo $realip_remote_addr $not_cloudflare {
        include /etc/nginx/include/not_cloudflare.conf;
    }
```

## nginx configuration in server context
```
    if ($not_cloudflare) { return 444; }
```

