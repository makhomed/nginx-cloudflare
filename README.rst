nginx-cloudflare (version 1.2.0)
================================

Set client real IP for nginx behind cloudflare.

Since Cloudflare acts as a reverse proxy, all connections come from one of Cloudflare's IP addresses. Cloudflare includes the originating IP address in the CF-Connecting-IP header.

This script, enabled in the cron, allows automatically maintaining an up-to-date `list of cloudflare ip addresses <https://www.cloudflare.com/ips/>`_ and reloading nginx if these addresses change.

First block of generated files, ``/etc/nginx/include/cloudflare.conf``, ``/etc/nginx/include/cloudflare_ipv4.conf``, ``/etc/nginx/include/cloudflare_ipv6.conf``, contains settings for nginx realip module, allowing nginx to restore real client IP from the CF-Connecting-IP header.

Second block of generated files, ``/etc/nginx/include/from_cloudflare.conf``, ``/etc/nginx/include/from_cloudflare_ipv4.conf``, ``/etc/nginx/include/from_cloudflare_ipv6.conf``, contains settings for nginx realip module, but only ``set_real_ip_from`` directives, allowing to manually configure ``real_ip_header`` directive to use ``X-Forwarded-For`` header from Cloudflare or ``True-Client-IP`` header from Cloudflare or ``CF-Connecting-IP`` header from Cloudflare, or any other future header, added by Cloudflare to proxied request from client.

Third block of generated file, ``/etc/nginx/include/not_cloudflare.conf``, ``/etc/nginx/include/not_cloudflare_ipv4.conf``, ``/etc/nginx/include/not_cloudflare_ipv6.conf``, contains settings for nginx geo module, allow blocking all non-cloudflare IPs in server context, blocking direct access to site, if anyone will try to bypassing Cloudflare. Suck blocking at nginx level may be required, if `nftables-tool <https://github.com/makhomed/nftables-tool/>`_ is not used, and nftables permits both types of connections - for some sites - direct connections from clients, and for some sites - only proxied connections via IP addresses from Cloudflare networks. In this case - ``not_cloudflare.conf`` can help to permit only requests from Cloudflare IPs for some sites with minimal overhead.

Installation
------------

- ``cd /opt ; git clone https://github.com/makhomed/nginx-cloudflare.git``
- ``yum install python3 python3-pip``
- ``python -m pip install --upgrade-strategy eager --upgrade Jinja2 requests invoke``

Upgrade
-------

- ``cd /opt/nginx-cloudflare ; git pull``
- ``python -m pip install --upgrade-strategy eager --upgrade Jinja2 requests invoke``

Usage
-----

.. code-block:: none

    /opt/nginx-cloudflare/nginx-cloudflare

or

.. code-block:: none

    /opt/nginx-cloudflare/nginx-cloudflare 172.21.0.0/16 2001:DB8:1111:2222::/64 2001:DB8:9999:7777::/64

Automation via cron
-------------------

Configure cron job, for example, in file ``/etc/cron.d/nginx-cloudflare``:

.. code-block:: none

    RANDOM_DELAY=360
    0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare

or

.. code-block:: none

    RANDOM_DELAY=360
    0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare 172.21.0.0/16 2001:DB8:1111:2222::/64 2001:DB8:9999:7777::/64

nginx configuration in http context
-----------------------------------

.. code-block:: none

    include /etc/nginx/include/cloudflare.conf;

    geo $realip_remote_addr $not_cloudflare {
        include /etc/nginx/include/not_cloudflare.conf;
    }

nginx configuration in server context
-------------------------------------

.. code-block:: none

    if ($not_cloudflare) { return 444; }

