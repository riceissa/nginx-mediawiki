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

## Contents

- [Set up the system](#set-up-the-system)
- [Set up nginx, php-fpm, and MySQL](#set-up-nginx-php-fpm-and-mysql)
- [DNS stuff](#dns-stuff)
- [Install MediaWiki](#install-mediawiki)
- [Set up HTTPS support](#set-up-https-support)
- [MediaWiki extensions](#mediawiki-extensions)
- [MediaWiki imports](#mediawiki-imports)
- [Interwiki links](#interwiki-links)

## Set up the system

- Follow [Getting Started with Linode](https://www.linode.com/docs/getting-started)
- Follow [Securing Your Server](https://www.linode.com/docs/security/securing-your-server)

As root:

```bash
apt update
apt upgrade
apt install lynx # useful later for configuring MediaWiki
```

## Set up nginx, php-fpm, and MySQL

- Reference [LEMP Server on Ubuntu 16.04 (Xenial Xerus)](https://www.linode.com/docs/websites/lemp/lemp-server-on-ubuntu-16-04);
  not all of it was necessary
- Reference [How To Install Linux, Nginx, MySQL, PHP (LEMP stack) in Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-in-ubuntu-16-04);
  not all of it was necessary

As root:

```bash
apt install nginx
apt install php-fpm php-mysql php-mbstring php-xml
systemctl restart php7.0-fpm
apt install mysql-server
mysql_secure_installation
```

Configure MySQL:

    mysql -u root -p
    create database timelines;
    # do more stuff

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

TODO talk about permissions for the files in the mediawiki archive.

## Set up HTTPS support

Instructions are modified from the [official instructions](https://certbot.eff.org/all-instructions/#ubuntu-16-04-xenial-nginx).

As root:

```bash
apt install letsencrypt
letsencrypt certonly --webroot -w /var/www/timelines -d timelines.issarice.com
letsencrypt renew --dry-run --agree-tos
vim /etc/crontab # add '0 0 1 * * root letsencrypt renew'
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
MobileFrontend is to make the appearance better on mobile devices, but is not
required.

The setup for these is pretty straightforward: download the tarball, extract it
to the correct location, and add a few lines in `LocalSettings.php`.
I'll just link to the official documentation because I have nothing to add.

- [Scribunto](https://www.mediawiki.org/wiki/Extension:Scribunto)
- [ParserFunctions](https://www.mediawiki.org/wiki/Extension:ParserFunctions)
- [Cite](https://www.mediawiki.org/wiki/Extension:Cite)
- [Google Analytics Integration](https://www.mediawiki.org/wiki/Extension:Google_Analytics_Integration)
- [MobileFrontend](https://www.mediawiki.org/wiki/Extension:MobileFrontend)
- Echo
- Thanks
- Interwiki (bundled with default install so no need to download separately)

## MediaWiki imports

[MediaWiki's instructions](https://www.mediawiki.org/wiki/Help:Templates#Copying_from_one_wiki_to_another)
specifically say to 'Uncheck the box "Include only the current revision, not the full history".'
**However** I find that unchecking this returns a very old version of the page for at least one
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
Template:Microsoft
Template:See also
Template:Yahoo! Inc.
Template:Sfn
Template:Update inline
Template:Verify source
Template:Dead link
Template:Cbignore
Template:US$
Template:Redirect
Template:Amazon.com
Template:Tesla Motors
Template:1/2
Template:1/4
Template:Efn-ua
Template:Interlanguage link
Template:Key press
Template:Notelist-ua
Template:Cite patent
Template:Sic
Template:ISSN
Template:US patent
Template:Cite newsgroup
Template:Timelines of computing
Template:Cn
Template:Harvnb
Template:Cancer timeline
Template:Authority control
Template:Inconsistent citations
Template:Influenza
Template:Portalbar
Template:Respiratory pathology
Template:Viral diseases
Template:Quote
Template:Citation needed
Template:Cite encyclopedia
Template:USD
Template:E-commerce
Template:Citation
Template:End div col
Template:Single double
Template:'"
Template:Double single
Template:"'
Template:PD
```

Here is an example of an importing process:

```bash
cd /var/www/timelines/maintenance
php importDump.php < /home/issa/Wikipedia-20170312181023.xml
php rebuildrecentchanges.php
```

## Interwiki links

**NOTE**: Newer versions of MediaWiki come with the [Interwiki extension](https://www.mediawiki.org/wiki/Extension:Interwiki).
You will need to add two lines to `LocalSettings.php` after which you can add
interwiki links from Special:Interwiki/add.
Alternatively use the database instructions below.

You will need to log in to the database for this.

The [example on the MediaWiki documentation](https://www.mediawiki.org/wiki/Manual:Interwiki#Single_line)
is slightly outdated.

Log in:

```bash
mysql -u your_username -p
# enter password
```

From the MySQL prompt, add the interwiki links:

```mysql
insert into interwiki (iw_prefix, iw_url, iw_local, iw_trans, iw_api, iw_wikiid)
values
('w','https://en.wikipedia.org/wiki/$1',1,0,'',''),
('devec','https://devec.subwiki.org/wiki/$1',1,0,'',''),
('market','https://market.subwiki.org/wiki/$1',1,0,'',''),
('demography','https://demography.subwiki.org/wiki/$1',1,0,'','');
```

Other wikis to consider:

- LessWrong Wiki
- Bitcoin Wiki (which one?)

## Upgrading to Ubuntu 17.10

When I originally wrote this page, the latest version of Ubuntu was 16.10. The
latest version now is Ubuntu 17.10. One difficulty with the upgrade here is that
MediaWiki 1.28 (or one of the extensions) is incompatible with php7.1-fpm,
which comes with Ubuntu 17.10. To
get around this, one must upgrade to MediaWiki 1.29.2.

The MediaWiki upgrade process was rather straightforward. I refereed to the following
references while performing the upgrade:

- [Vipul Naik's upgrade steps](https://raw.githubusercontent.com/vipulnaik/working-drafts/dd0b3bc108789a5d16d9f03127165a935267aa7e/install-update-and-recover/mediawiki-1.29.txt)
- [Manual:Upgrading](https://www.mediawiki.org/wiki/Manual:Upgrading)

For juggling the directories in `/var/www`, I would did the following:

- First, stop nginx
- Set up the new MediaWiki directory in `/var/www/timelines-new`
- Move `/var/www/timelines` to `/var/www/timelines-old`
- Move `/var/www/timelines-new` to `/var/www/timelines`

I copied `LocalSettings.php` and `images/`. I did not copy over
any extensions, choosing instead to just install them again.
