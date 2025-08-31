# nginx-cloudflare (version 2.0.0)

Sets the client's real IP in nginx behind Cloudflare.

Since Cloudflare acts as a reverse proxy, all connections appear to come from Cloudflare IP addresses. Cloudflare provides the original client IP address in the [X-Forwarded-For](https://developers.cloudflare.com/fundamentals/reference/http-headers/#x-forwarded-for) and [CF-Connecting-IP](https://developers.cloudflare.com/fundamentals/reference/http-request-headers/#cf-connecting-ip) headers. This script, when enabled in the cron, automatically maintains an up-to-date [list of Cloudflare IP addresses](https://www.cloudflare.com/ips/) and reloads nginx if these addresses are changed after update.

The file `/etc/nginx/include/cloudflare_set_realip_from.conf` contains a generated set of [set_real_ip_from](https://nginx.org/en/docs/http/ngx_http_realip_module.html#set_real_ip_from) directives for Cloudflare's IP ranges. The file `/etc/nginx/include/cloudflare_not_deny_access.conf` contains a generated fragment for the [nginx geo module](https://nginx.org/en/docs/http/ngx_http_geo_module.html), which allows nginx to block all non-Cloudflare IP addresses, using the `if` directive from the [nginx rewrite module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html).

## Installation

> [!IMPORTANT]
> Python 3.9+ and the [Jinja2](https://jinja.palletsprojects.com/), [requests](https://requests.readthedocs.io/), and [invoke](https://www.pyinvoke.org/) modules are required.

```bash
dnf -y install python3 python3-pip ; \
python3 -m pip install --no-input --upgrade-strategy eager --upgrade Jinja2 requests invoke ; \
cd /opt ; git clone https://github.com/makhomed/nginx-cloudflare.git
```

## Upgrade

```bash
python3 -m pip install --no-input --upgrade-strategy eager --upgrade Jinja2 requests invoke ; \
cd /opt/nginx-cloudflare ; git pull
```

## Usage

```bash
/opt/nginx-cloudflare/nginx-cloudflare
```

## Automation

Configure a cron job, for example, in the file `/etc/cron.d/nginx-cloudflare`:

```cron
RANDOM_DELAY=60

0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare
```

## nginx configuration in `http` context

Most simple configuration, when only Cloudflare is used:

```nginx
include /etc/nginx/include/cloudflare_set_realip_from.conf;
real_ip_header CF-Connecting-IP;
```

More complex configuration, when multiple CDN / DDoS protection services are used:

```nginx
include /etc/nginx/include/cloudflare_set_realip_from.conf;
include /etc/nginx/include/other_cdn_set_realip_from.conf;
include /etc/nginx/include/other_service_set_realip_from.conf;
real_ip_header X-Forwarded-For;
real_ip_recursive on;
```

If the `$request_not_from_cloudflare` variable is needed, create it:


```nginx
geo $realip_remote_addr $request_not_from_cloudflare {
    include /etc/nginx/include/cloudflare_not_deny_access.conf;
}
```

## nginx configuration in `server` context

```nginx
if ($request_not_from_cloudflare) { return 444; }
```

