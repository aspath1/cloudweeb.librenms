Ansible Role LibreNMS
=========

[![Build Status](https://travis-ci.com/cloudweeb/cloudweeb.librenms.svg?branch=master)](https://travis-ci.com/cloudweeb/cloudweeb.librenms)

Ansible role to install LibreNMS on Linux Server

Requirements
------------

None

Role Variables
--------------

| Variable                   | Default Value       | Description                        |
|----------------------------|---------------------|------------------------------------|
| librenms_user              | librenms            | LibreNMS system user               |
| librenms_group             | {{ librenms_user }} | LibreNMS system group              |
| librenms_web_user          | apache              | LibreNMS web server user           |
| librenms_home_dir          | /opt/librenms       | Path to LibreNMS base dir          |
| librenms_db_host           | localhost           | LibreNMS db host                   |
| librenms_db_name           | librenms            | LibreNMS db name                   |
| librenms_db_user           | librenms            | LibreNMS db user                   |
| librenms_db_pass           | librenms            | LibreNMS db password               |
| librenms_admin_acct        | []                  | List of LibreNMS users             |
| librenms_required_packages | []                  | List of LibreNMS required packages |

Dependencies
------------

- role: geerlingguy.repo-epel
- role: geerlingguy.apache / geerlingguy.nginx
- role: cloudweeb.php
- role: cloudweeb.mariadb

Example Playbook
----------------

LibreNMS + LAMP

    - hosts: servers
      vars:

        librenms_db_pass: "{{ lookup('password',
                                '/tmp/passwordfile
                                chars=ascii_letters,digits,hexdigits,punctuation'
                                ) }}"

        apache_vhosts:
          - servername: librenms.example.com
              documentroot: /opt/librenms/html/
              extra_parameters: |
              AllowEncodedSlashes NoDecode

        php_version: 7.2

        php_extra_packages:
          - composer
          - php-mysqlnd
          - php-curl
          - php-gd
          - php-mbstring
          - php-process
          - php-snmp
          - php-xml
          - php-zip

        mariadb_root_password: "{{ lookup('password', '/tmp/mysql_root_password
                                        length=15
                                        chars=ascii_letters,digits,hexdigits') }}"

      roles:

        - geerlingguy.repo-epel
        - geerlingguy.apache
        - cloudweeb.php
        - cloudweeb.mariadb
        - cloudweeb.librenms

LibreNMS + LEMP

---

    - hosts: servers
      vars:

        librenms_db_pass: "{{ lookup('password',
                                '/tmp/passwordfile
                                chars=ascii_letters,digits,hexdigits,punctuation'
                                ) }}"

        nginx_vhosts:
          - listen: "80"
            server_name: "nms.example.com"
            root: "/opt/librenms/html"
            index: "index.php index.html index.htm"
            state: "present"
            template: "{{ nginx_vhost_template }}"
            extra_parameters: |
              location / {
                try_files $uri $uri/ /index.php?$query_string;

                location ~* ^.+\.(jpeg|jpg|png|gif|bmp|ico|svg|css|js)$ {
                    expires     max;
                }

                location ~ [^/]\.php(/|$) {
                    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                    if (!-f $document_root$fastcgi_script_name) {
                        return  404;
                    }

                    fastcgi_pass    unix:/var/run/php-librenms.sock;
                    fastcgi_index   index.php;
                    include         /etc/nginx/fastcgi_params;
                }

              }
              location /api/v0 {
                try_files $uri $uri/ /api_v0.php?$query_string;
              }
              location ~ /\.ht {
                deny all;
              }

        php_version: 7.2
        php_fpm_enabled: true

        php_extra_packages:
          - composer
          - php-mysqlnd
          - php-curl
          - php-gd
          - php-mbstring
          - php-process
          - php-snmp
          - php-xml
          - php-zip

        mariadb_root_password: "{{ lookup('password', '/tmp/mysql_root_password
                                        length=15
                                        chars=ascii_letters,digits,hexdigits') }}"

      roles:

        - geerlingguy.repo-epel
        - geerlingguy.nginx
        - cloudweeb.php
        - cloudweeb.mariadb
        - cloudweeb.librenms


License
-------

BSD/MIT

Author Information
------------------

Agnesius Santo Naibaho
