nginx-cloudflare (version 1.0.0)
================================

Set client real ip for nginx behind cloudflare.

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

    geo $not_cloudflare {
        include /etc/nginx/include/not_cloudflare.conf;
    }

nginx configuration in server context
-------------------------------------

.. code-block:: none

    if ($not_cloudflare) { return 444; }

