# drupal-installation-issues
## What's Drupal? 
Drupal is a free and open-source web content management system written in PHP and distributed under the GNU General Public License. Drupal provides a back-end framework for at least 13% of the top 10,000 websites worldwide – ranging from personal blogs to corporate, political, and government sites. 
```
Stable release: 9.1.6 / 2021-04-07
Initial release: January 15, 2001; 20 years ago
Developer(s): Drupal community
Operating system: Unix-like, Windows
Platform: Web platform
Written in: PHP, using Symfony
Original author: Dries Buytaert
```
## Why should use Drupal?
Drupal is a enterprise level solution for business applications. It can be used for complicated business requirement implementation and easy to be extended. Provide a standard UI and data model constructor. all modules in Drupal can be managed like a program in operation system. https://www.drupal.org/docs/develop
## How it works with Java technologies?
Drupal provided a really easy way to integrate with different platform, support many kinds of standard protocol. you also can integrate it with Java APIs as well.
## How to install Drupal?
You can follow the material in drupal.org. really rich documentation in internet can be found. Here are a bunch of documents for you: https://www.drupal.org/docs/develop/local-server-setup/linux-development-environments
## How could I install it with nginx?
You need install [php-fpm](https://php-fpm.org/) modules to integrate with nginx. https://www.nginx.com/resources/wiki/start/topics/recipes/drupal/
```
server {
    server_name example.com;
    root /var/www/drupal8; ## <-- Your only path reference.

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # Very rarely should these ever be accessed outside of your lan
    location ~* \.(txt|log)$ {
        allow 192.168.0.0/16;
        deny all;
    }

    location ~ \..*/.*\.php$ {
        return 403;
    }

    location ~ ^/sites/.*/private/ {
        return 403;
    }

    # Block access to scripts in site files directory
    location ~ ^/sites/[^/]+/files/.*\.php$ {
        deny all;
    }

    # Allow "Well-Known URIs" as per RFC 5785
    location ~* ^/.well-known/ {
        allow all;
    }

    # Block access to "hidden" files and directories whose names begin with a
    # period. This includes directories used by version control systems such
    # as Subversion or Git to store control files.
    location ~ (^|/)\. {
        return 403;
    }

    location / {
        # try_files $uri @rewrite; # For Drupal <= 6
        try_files $uri /index.php?$query_string; # For Drupal >= 7
    }

    location @rewrite {
        #rewrite ^/(.*)$ /index.php?q=$1; # For Drupal <= 6
        rewrite ^ /index.php; # For Drupal >= 7
    }

    # Don't allow direct access to PHP files in the vendor directory.
    location ~ /vendor/.*\.php$ {
        deny all;
        return 404;
    }

    # Protect files and directories from prying eyes.
    location ~* \.(engine|inc|install|make|module|profile|po|sh|.*sql|theme|twig|tpl(\.php)?|xtmpl|yml)(~|\.sw[op]|\.bak|\.orig|\.save)?$|/(\.(?!well-known).*)|Entries.*|Repository|Root|Tag|Template|composer\.(json|lock)|web\.config$|/#.*#$|\.php(~|\.sw[op]|\.bak|\.orig|\.save)$ {
        deny all;
        return 404;
    }

    # In Drupal 8, we must also match new paths where the '.php' appears in
    # the middle, such as update.php/selection. The rule we use is strict,
    # and only allows this pattern with the update.php front controller.
    # This allows legacy path aliases in the form of
    # blog/index.php/legacy-path to continue to route to Drupal nodes. If
    # you do not have any paths like that, then you might prefer to use a
    # laxer rule, such as:
    #   location ~ \.php(/|$) {
    # The laxer rule will continue to work if Drupal uses this new URL
    # pattern with front controllers other than update.php in a future
    # release.
    location ~ '\.php$|^/update.php' {
        fastcgi_split_path_info ^(.+?\.php)(|/.*)$;
        # Ensure the php file exists. Mitigates CVE-2019-11043
        try_files $fastcgi_script_name =404;
        # Security note: If you're running a version of PHP older than the
        # latest 5.3, you should have "cgi.fix_pathinfo = 0;" in php.ini.
        # See http://serverfault.com/q/627903/94922 for details.
        include fastcgi_params;
        # Block httpoxy attacks. See https://httpoxy.org/.
        fastcgi_param HTTP_PROXY "";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param QUERY_STRING $query_string;
        fastcgi_intercept_errors on;
        # PHP 5 socket location.
        #fastcgi_pass unix:/var/run/php5-fpm.sock;
        # PHP 7 socket location.
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        try_files $uri @rewrite;
        expires max;
        log_not_found off;
    }

    # Fighting with Styles? This little gem is amazing.
    # location ~ ^/sites/.*/files/imagecache/ { # For Drupal <= 6
    location ~ ^/sites/.*/files/styles/ { # For Drupal >= 7
        try_files $uri @rewrite;
    }

    # Handle private files through Drupal. Private file's path can come
    # with a language prefix.
    location ~ ^(/[a-z\-]+)?/system/files/ { # For Drupal >= 7
        try_files $uri /index.php?$query_string;
    }

    # Enforce clean URLs
    # Removes index.php from urls like www.example.com/index.php/my-page --> www.example.com/my-page
    # Could be done with 301 for permanent or other redirect codes.
    if ($request_uri ~* "^(.*/)index\.php/(.*)") {
        return 307 $1$2;
    }
}
```
## How could I use Oracle as Drupal database?
You need to install the oracle driver first under the drupal root folder. 
```
composer require 'drupal/oracle:1.x-dev@dev'
composer install
```
so before you execute the command, you need to install composer first.
and also you need to copy the driver folder under DRUPAL_ROOT, as there is a logic to validate the folder and scan for drivers: `DRUPAL_ROOT . '/drivers/lib/Drupal/Driver/Database`
## How to install Composer?
Please see the documentation: https://getcomposer.org/download/
## What's difference after I install the drupal oracle driver?
You will see a new database type in the database installation page.
## Why I could not see Oracle database type from the installation page?
There is a logic in drupal core to validate pdo_oci module in PHP. If you have installed it, you would be able to see the Oracle database type.
## How could I install pdo_oci module?
- Install the [InstantClient](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html) from RPMs. You need install basic package, developement and runtime package as well.
- Download the PHP sources, matching the current PHP version installed php -v
```
PHP 7.3.29 (cli) (built: Jan 18 2020 13:49:07) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
```
- https://github.com/php/php-src/tags
```
cd /root
mkdir src
cd src
wget https://github.com/php/php-src/archive/refs/tags/php-7.3.29.tar.gz
tar xfvz php-7.3.29.tar.gz
```
- Compile and install the extension
```
cd php-7.3.29
cd ext
cd pdo_oci
phpize
./configure --with-pdo-oci=instantclient,[YOUR_ORACLE_LIBRARY_PATH],21
make
make install
echo extension=pdo_oci.so > /etc/php.d/pdo_oci.ini
php -i | grep oci
```
## Error: ORA-00955: name is already used by an existing object
Please refer to the article: https://www.ibm.com/support/pages/error-ora-00955-name-already-used-existing-object
## ORA-00972: identifier is too long
Database object names in 11g as well as in 12cR1 are limited to 30 bytes (in a single-byte character set it will be equivalent to 30 characters).
The 30 bytes object names restriction has been lifted in second release of Oracle Database 12c (12cR2) and if the value of the COMPATIBLE initialization parameter is set to 12.2 or higher then object names' length can be up to 128 bytes.
If you could not change Oracle version, you can change the CONSTANTS in Oracle driver (Connection.php) to be: `define('ORACLE_IDENTIFIER_MAX_LENGTH', 30);`
