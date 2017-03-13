# nginx-mediawiki

Instructions and sample configuration for setting up MediaWiki on nginx on Ubuntu 16.10

These instructions were written in the process of setting up the
[timelines wiki](https://timelines.issarice.com/wiki/Main_Page).

Full list of software:

- Ubuntu 16.10
- nginx
- MediaWiki
- MediaWiki extensions
- MediaWiki templates
- php7.0-fpm
- certbot (letsencrypt)
- Linode for domain stuff

## Set up the system

- Follow [Getting Started with Linode](https://www.linode.com/docs/getting-started)
- Follow [Securing Your Server](https://www.linode.com/docs/security/securing-your-server)
- Reference [LEMP Server on Ubuntu 16.04 (Xenial Xerus)](https://www.linode.com/docs/websites/lemp/lemp-server-on-ubuntu-16-04)

As root:

```bash
apt update
apt upgrade
apt install nginx
apt install mysql-server
mysql_secure_installation
apt install php-fpm php-mysql
systemctl restart php7.0-fpm
apt install lynx # useful later for configuring MediaWiki
apt install php-mbstring php-xml
```

## Set up nginx

coming soon.

## Set up MySQL

coming soon.

## DNS stuff

On Linode's DNS manager, add a new subdomain for `timelines.issarice.com`, and
make sure to point it to the new instance.
Also on the new Linode instance, set up reverse DNS.
Note that **even if you do everything right, there will be a delay until the
domain starts working**, so don't panic.

## Install MediaWiki

We will be installing MediaWiki at `/var/www/timelines/`.
Modified from the [instructions on the MediaWiki manual](https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu)
(note that the official Ubuntu guide is for Apache, and here we're using nginx):

```bash
mkdir /home/issa/Downloads
cd /home/issa/Downloads
wget https://releases.wikimedia.org/mediawiki/1.28/mediawiki-1.28.0.tar.gz
tar xvzf mediawiki-1.28.0.tar.gz
mv mediawiki-1.28.0 /var/www/timelines
```

Now do `lynx http://localhost/index.php` to fill in the web form to complete
the installation.
The [setup instructions on the MediaWiki site](https://www.mediawiki.org/wiki/Manual:Config_script)
are pretty detailed, and it pretty much goes as expected.

## Set up HTTPS support

Instructions are modified from the [official instructions](https://certbot.eff.org/all-instructions/#ubuntu-16-04-xenial-nginx).

As root:

```bash
apt install letsencrypt
letsencrypt certonly --webroot -w /var/www/timelines -d timelines.issarice.com
letsencrypt renew --dry-run --agree-tos
vim /etc/crontab # add '0 0 1 * * letsencrypt renew'
vim /etc/nginx/sites-available/default
```

Now in `/etc/nginx/sites-available/default` add a new block to redirect all
HTTP connections to HTTPS:

    server {
            listen 80;
            listen [::]:80;
            server_name timelines.issarice.com;
            return 301 https://$host$request_uri;
    }

In the regular server block, change the port and give an SSL certificate:

    server {
            listen 443 ssl default_server;
            listen [::]:443 ssl default_server;
            ssl_certificate /etc/letsencrypt/live/timelines.issarice.com/fullchain.pem;
            ssl_certificate_key /etc/letsencrypt/live/timelines.issarice.com/privkey.pem;

            ...
    }

## MediaWiki extensions

The first three of the following MediaWiki extensions are required for correctly rendering
wikitext copied from the English Wikipedia.
This is because the English Wikipedia uses various templates and extra tags.
Google Analytics is a personal preference but is not required for rendering
content.

The setup for these is pretty straightforward: download the tarball, extract it
to the correct location, and add a few lines in `LocalSettings.php`.
I'll just link to the official documentation because I have nothing to add.

- [Scribunto](https://www.mediawiki.org/wiki/Extension:Scribunto)
- [ParserFunctions](https://www.mediawiki.org/wiki/Extension:ParserFunctions)
- [Cite](https://www.mediawiki.org/wiki/Extension:Cite)
- [Google Analytics Integration](https://www.mediawiki.org/wiki/Extension:Google_Analytics_Integration)

## MediaWiki imports

[MediaWiki's instructions](https://www.mediawiki.org/wiki/Help:Templates#Copying_from_one_wiki_to_another)
specifically say to 'Uncheck the box "Include only the current revision, not the full history".'
**However** I find that checking this returns a very old version of the page for at least one
page (`MediaWiki:Common.css`).
I have no idea why this is, but one remedy is to export both with this box checked and with this
box unchecked.
You can always run `php rebuildrecentchanges.php` in the `maintenance` directory to
deduplicate revisions, so I'm pretty sure importing twice is fine.
(Note: I began by trying to import the XML dump with the "full history" so I have
no idea what happens if you only try to get the most recent version.)

The MediaWiki instructions linked above is fine for step 1.
But since we're exporting from the English Wikipedia, go to [Special:Export](https://en.wikipedia.org/wiki/Special:Export)
on that wiki instead.
As for step 2, I found Special:Import to be unreliable because the connection
kept resetting, so I used `importDump.php` in the MediaWiki `maintenance`
directory instead.

List of pages I exported:

```
Template:Reflist
Template:Snd
Template:Spaced en dash
Template:Technology company timelines
Template:Navbox
MediaWiki:Common.css
MediaWiki:Common.js
Template:Column-width
Template:Cite journal
Template:Cite web
Template:Cite press release
Template:Cite book
Template:Use mdy dates
Template:Cite conference
Template:Rp
Template:As of
MediaWiki:Gadget-ReferenceTooltips.css
MediaWiki:Gadget-ReferenceTooltips.js
```

Here is an example of an importing process:

```bash
cd /var/www/timelines/maintenance
php importDump.php < /home/issa/Wikipedia-20170312181023.xml
php rebuildrecentchanges.php
```
