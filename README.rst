nginx-cloudflare (version 1.0.0)
================================

Set client real ip for nginx behind cloudflare.

Since CloudFlare acts as a reverse proxy, all connections come from one of CloudFlare's IP addresses. CloudFlare includes the originating IP address in the CF-Connecting-IP header.

This script, enabled in the cron, allow automatically maintain up-to-date `list of cloudflare ip addresses <https://www.cloudflare.com/ips/>`_ and reload nginx if these addresses changes.

First generated file, ``/etc/nginx/include/cloudflare.conf``, contains settings for nginx realip module, allow nginx restore real client ip from the CF-Connecting-IP header.

Second generated file, ``/etc/nginx/include/not_cloudflare.conf``, contains settings for nginx geo module, allow blocking all non-cloudflare ips in server context, blocking direct access to site, if anyone will try to bypassing cloudflare.

Installation
------------

- ``cd /opt``
- ``git clone https://github.com/makhomed/nginx-cloudflare.git nginx-cloudflare``

Also you need to install ``python3``.

Upgrade
-------

- ``cd /opt/nginx-cloudflare``
- ``git pull``

Usage
-----

.. code-block:: none

    /opt/nginx-cloudflare/nginx-cloudflare

Automation via cron
-------------------

Configure cron job, for example, in file ``/etc/cron.d/nginx-cloudflare``:

.. code-block:: none

    RANDOM_DELAY=360

    0 0 * * * root /opt/nginx-cloudflare/nginx-cloudflare

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

