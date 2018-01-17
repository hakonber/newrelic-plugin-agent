NewRelic Plugin Agent
=====================

| An agent that polls a number of backend systems and submits the results to the NewRelic platform. The backend systems supported by the original repo can be found `here <https://github.com/MeetMe/newrelic-plugin-agent>`_.
|
| This fork was created to initially only allow collection of Apache HTTP server metrics with Python 3.x (not backwards compatible with Python 2.x). As such, other backend system plugins in this fork may not run entirely and this README file simplified until such time those plugins are supported.

Base Requirements
-----------------
The agent requires ``pip3`` for installation. This fork is tested and known to work on Python 3.5. It won't work on Python 2.x and mileage may vary on other Python 3.x.

Configuration File Note
-----------------------
The configuration file uses YAML as its format. Most tickets for non-working installs are due to configuration file formatting errors. Please make sure you are properly formatting your configuration file prior to submitting a ticket. YAML is a whitespace dependent markup format. More information on writing proper YAML can be found at http://yaml.org.

Installation Instructions
-------------------------
Note: This fork currently only supports installation via the GitHub repo.

1. Install via ``pip3``:

::

    $ pip3 install git+git://github.com/SuperGeno/newrelic-plugin-agent.git@{ branch_or_tag_or_commit }

* See ``pip3`` installation instructions at https://pip.pypa.io/en/latest/installing/

2. Copy the configuration file example from ``/opt/newrelic-plugin-agent/newrelic-plugin-agent.cfg`` to ``/etc/newrelic/newrelic-plugin-agent.cfg`` and edit the configuration in that file.

3. Make a ``/var/log/newrelic`` directory and make sure it is writable by the user specified in the configuration file

4. Make a ``/var/run/newrelic`` directory and make sure it is writable by the user specified in the configuration file

5. Run the app:

::

    $ newrelic-plugin-agent -c PATH-TO-CONF-FILE [-f]

Where ``-f`` is to run it in the foreground instead of as a daemon.

Sample configuration and init.d scripts are installed to ``/opt/newrelic-plugin-agent`` in addition to a PHP script required for APC monitoring.

Plugin Configuration Stanzas
----------------------------
Each plugin can support gathering data from a single or multiple targets. To support multiple targets for a plugin, you create a list of target stanzas:

::

    plugin_name:
      - name: target_name
        host: localhost
        foo: bar
      - name: target_name
        host: localhost
        foo: bar

While you can use the multi-target format for a plugin's configuration stanza like:

::

    plugin_name:
      - name: target_name
        host: localhost
        foo: bar

You can also use a single mapping like follows:

::

    plugin_name:
      name: target_name
      host: localhost
      foo: bar

The fields for plugin configurations can vary due to a plugin's configuration requirements. The name value in each stanza is only required when using multiple targets in a plugin. If it is only a single target, the name will be taken from the server's hostname.

Apache HTTPd Installation Notes
-------------------------------
Enable the HTTPd server status page in the default virtual host. The following example configuration snippet for Apache HTTPd 2.2 demonstrates how to do this:

::

    <Location /server-status>
        SetHandler server-status
        Order deny,allow
        Deny from all
        Allow from 127.0.0.1
    </Location>

For HTTPd 2.4, it should look something like:

::

    <Location /server-status>
        SetHandler server-status
        Require ip 127.0.0.1
    </Location>

The agent requires the extended information to parse metrics. If you are not seeing any metrics on your graphs for Apache verify that you have enabled ``ExtendedStatus``, the default is off so you must enable it. In your global Apache HTTP configuration you need to enable exetended status using:

::

    ExtendedStatus On

If you are monitoring Apache HTTPd via a HTTPS connection you can use the ``verify_ssl_cert`` configuration value in the httpd configuration section to disable SSL certificate verification.

Configuration Example
---------------------

::

    %YAML 1.2
    ---
    Application:
      license_key: REPLACE_WITH_REAL_KEY
      poll_interval: 60
      #newrelic_api_timeout: 10
      #proxy: http://localhost:8080

      apache_httpd:
         -  name: hostname1
            scheme: http
            host: localhost
            port: 80
            path: /server-status
            #verify_ssl_cert: true
         -  name: hostname2
            scheme: http
            host: localhost
            port: 80
            path: /server-status
            #verify_ssl_cert: true

      couchdb:
         -  name: localhost
            host: localhost
            port: 5984
            #verify_ssl_cert: true
            #username: foo
            #password: bar
         -  name: localhost
            host: localhost
            port: 5984
            #verify_ssl_cert: true
            #username: foo
            #password: bar

      elasticsearch:
        name: clustername
        host: localhost
        port: 9200

      haproxy:
        name: my-haproxy-server
        host: localhost
        port: 80
        path: /haproxy?stats;csv
        scheme: http
        #verify_ssl_cert: true
        #username: foo
        #password: bar

      mongodb:
        name: hostname
        host: localhost
        port: 27017
        admin_username: foo
        admin_password: bar
        databases:
          database_name_1:
            username: foo
            password: bar
          database_name_2:
            username: foo
            password: bar

      memcached:
        - name: localhost
          host: localhost
          port: 11211
          path: /path/to/unix/socket
        - name: localhost
          host: localhost
          port: 11211
          path: /path/to/unix/socket

      nginx:
        - name: hostname
          host: localhost
          port: 80
          path: /nginx_stub_status
          #verify_ssl_cert: true
        - name: hostname
          host: localhost
          port: 80
          path: /nginx_stub_status
          #verify_ssl_cert: true

      pgbouncer:
        - host: localhost
          port: 6000
          user: stats

      php_apc:
         scheme: http
         host: localhost
         port: 80
         path: /apc-nrp.php
         #username: foo
         #password: bar
         #verify_ssl_cert: t

      php_fpm:
        - name: fpm-pool
          scheme: https
          host: localhost
          port: 443
          path: /fpm_status
          query: json

      postgresql:
        - host: localhost
          port: 5432
          user: postgres
          dbname: postgres
          superuser: True

      rabbitmq:
        - name: rabbitmq@localhost
          host: localhost
          port: 15672
          username: guest
          password: guest
          #verify_ssl_cert: true
          api_path: /api

      redis:
        - name: localhost
          host: localhost
          port: 6379
          db_count: 16
          password: foobar
          #path: /var/run/redis/redis.sock
        - name: localhost
          host: localhost
          port: 6380
          db_count: 16
          password: foobar
          #path: /var/run/redis/redis.sock

      riak:
        - name: localhost
          host: localhost
          port: 8098
          #verify_ssl_cert: true

    Daemon:
      user: newrelic
      pidfile: /var/run/newrelic/newrelic-plugin-agent.pid

    Logging:
      formatters:
        verbose:
          format: '%(levelname) -10s %(asctime)s %(process)-6d %(processName) -15s %(threadName)-10s %(name) -25s %(funcName) -25s L%(lineno)-6d: %(message)s'
      handlers:
        file:
          class : logging.handlers.RotatingFileHandler
          formatter: verbose
          filename: /var/log/newrelic/newrelic-plugin-agent.log
          maxBytes: 10485760
          backupCount: 3
      loggers:
        newrelic-plugin-agent:
          level: INFO
          propagate: True
          handlers: [console, file]
        requests:
          level: ERROR
          propagate: True
          handlers: [console, file]

Troubleshooting
---------------
- If the application installs but doesn't seem to be submitting status, check the logfile which at ``/tmp/newrelic-plugin-agent.log`` if the default example logging configuration is used.
- If the agent starts but dies shortly after ensure that ``/var/log/newrelic`` and ``/var/run/newrelic`` are writable by the same user specified in the daemon section of the configuration file.
- If the agent has died and won't restart, remove any files found in ``/var/run/newrelic/``
- If using the Apache HTTP plugin and your stats are blank, ensure the ExtendedStatus directive is on.
