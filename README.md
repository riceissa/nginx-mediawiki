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
Template:Reference page
Module:Citation/CS1/Suggestions
Template:Date table sorting
Template:Single+double
Template:Double+single
Template:Blockquote
```

Here is an example of an importing process:

```bash
cd /var/www/timelines/maintenance
php importDump.php < /home/issa/Wikipedia-20170312181023.xml
php rebuildrecentchanges.php
```

## Cite web template fix

As of 2025, one of cite web's dependencies, <https://en.wikipedia.org/wiki/Module:Citation/CS1/Configuration>,
uses external data that is stored on commons.wikimedia.org. By default, your MediaWiki installation
will not be able to access this data. However, in Module:Citation/CS1/Configuration this is a line that
looks like:

```lua
local use_commons_data = true; -- set to false if your wiki does not have access to mediawiki commons
```

Toggle this to `false` to use the hardcoded values in the Lua file. This will make Cite web work.

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

## Reference tooltips

[Reference tooltips](https://www.mediawiki.org/wiki/Reference_Tooltips) are the the hover-over popups that show up when you hover over a reference superscript. If you used the [MediaWiki imports above](#mediawiki-imports) then this will already be included. However if you didn't include reference tooltips in your imports, then this is another way to do the same thing.

To get reference tooltips for your wiki, here are the steps:

* Get the JavaScript: Go to https://en.wikipedia.org/w/index.php?title=MediaWiki:Gadget-ReferenceTooltips.js&action=edit and copy the source code to your wiki, by going to https://yourwiki.domain/index.php?title=MediaWiki:Gadget-ReferenceTooltips.js&action=edit
* Get the CSS: Go to https://en.wikipedia.org/w/index.php?title=MediaWiki:Gadget-ReferenceTooltips.css&action=edit and copy the source code to your wiki, by going to https://yourwiki.domain/index.php?title=MediaWiki:Gadget-ReferenceTooltips.css&action=edit
* Go to https://yourwiki.domain/index.php?title=MediaWiki:Common.js&action=edit and add the following lines, which will import the reference tooltips JavaScript/CSS so that it is loaded on every page:

  ```javascript
  mw.loader.load('/index.php?title=MediaWiki:Gadget-ReferenceTooltips.js&action=raw&ctype=text/javascript');
  mw.loader.load('/index.php?title=MediaWiki:Gadget-ReferenceTooltips.css&action=raw&ctype=text/css', 'text/css');
  ```

## Upgrading to Ubuntu 17.10

When I originally wrote this page, the latest version of Ubuntu was 16.10. The
latest version now is Ubuntu 17.10. One difficulty with the upgrade here is that
MediaWiki 1.28 (or one of the extensions) is incompatible with php7.1-fpm,
which comes with Ubuntu 17.10. To
get around this, I upgraded to MediaWiki 1.29.2 (along with all of the
extensions).

The MediaWiki upgrade process was rather straightforward. I referred to the following
references while performing the upgrade:

- [Vipul Naik's upgrade steps](https://raw.githubusercontent.com/vipulnaik/working-drafts/dd0b3bc108789a5d16d9f03127165a935267aa7e/install-update-and-recover/mediawiki-1.29.txt)
- [Manual:Upgrading](https://www.mediawiki.org/wiki/Manual:Upgrading)

For juggling the directories in `/var/www`, I did the following:

- Stop nginx
- Set up the new MediaWiki directory in `/var/www/timelines-new`
- Move `/var/www/timelines` to `/var/www/timelines-old`
- Move `/var/www/timelines-new` to `/var/www/timelines`

I copied `LocalSettings.php` and `images/`. I did not copy over
any extensions, choosing instead to just install them again.

The suggested commands for changing permissions of the `images/` directory
(`find ./images -type d -exec chmod 755 {} \;` and
`chgrp -R www-data images`) might not be enough.
I found that I was unable to upload any images ("Could not open lock file" etc.).
To fix this, I also had to run `chown -R www-data:www-data images`.

## Upgrading to Ubuntu 18.04

Ubuntu 18.04 has PHP 7.2 as the default,
but [MediaWiki does not yet support this version of
PHP](https://www.mediawiki.org/wiki/Compatibility#PHP).
It's supported starting MediaWiki version 1.31,
but 1.31 is apparently not yet an official release.

So what I did was keep the current version of MediaWiki
and use a PPA to install PHP 7.1 again:

- <https://serverfault.com/questions/895746/switch-from-php-7-2-to-7-1-on-ubuntu-16-04-apache>
- <https://websiteforstudents.com/downgrade-php-7-2-to-php-7-1-with-nginx-support-on-ubuntu-16-04-17-10-and-18-04/>

Then in the nginx config, I just specified that PHP 7.1 should be used (which
should have been the case already, since PHP 7.1 was the version before the
upgrade).

## Upgrading to Ubuntu 24.04

After many years of not touching the server at all except to deal with bot
attacks (oops), I finally sat down to upgrade to the latest Ubuntu LTS and to also
upgrade the MediaWiki to the latest.  This took a _lot_ of work over multiple days, so
I want to document some of that here.

### Ubuntu

The Ubuntu upgrades were pretty painless. It was mostly just:

```bash
sudo apt update
sudo apt upgrade
sudo do-release-upgrade
```

To go from 18.04 to 24.04, I had to do a chain of LTS upgrades:

18.04 → 20.04 → 22.04 → 24.04

For the first two upgrades, whenever I was prompted whether to keep my
current configuration files or to install the maintainer's new version,
I just selected to _keep my current files_.  For the final upgrade,
I decided I should review the diff. I went into the shell and ran vimdiff
to review the changes. I pulled in any new things that seemed good to have.

There were two problems after the Ubuntu upgrades: (1) the MW site
that used templates were completely broken (because the old Scribunto
couldn't run on the new PHP, or something like that), and (2) all my
pip packages were gone, in particular the one for AWS, so backups
were failing to upload.

Problem (2) was easy to fix, just:

```bash
sudo apt install python3-pip
pip3 install --break-system-packages awscli
```

Problem (1) was harder.

### MediaWiki upgrades from 1.29 to 1.44

MW upgrades proceed like this. You first cd to `/var/www` and then
download the tarball for the new MW release there.  Then you unpack it,
and move the `images` folder and `LocalSettings.php` from the old MW
folder to the new one. Then you run:

```bash
php maintenance/update.php
```

from the new MW folder. However, when I did this to try to upgrade
from 1.29 to 1.44, MW complained that 1.29 was too old ("Can not
upgrade from versions older than 1.35, please upgrade to that version
or later first."), and that I had to go to 1.35 first. So I ended up
repeating the whole thing again, making a 1.35 MW folder first, and
running the upgrade script there first. This worked, and then I went
back to the 1.44 folder and ran the upgrade script there.

The upgrade from 1.35 to 1.44 kept failing; the error was "Error: 2006
MySQL server has gone away". This kept happening, and finally what
worked was to shut down the nginx process and _then_ rerun the upgrade
php script. My guess is that the server was already taxed from having
to deal with requests being made to it (from visitors and bots) and so
couldn't also do the upgrade. However, since my server has a cronjob
to restart nginx if it detects nginx has stopped, I had to first
disable that cronjob (and then not forget to turn it back on once the
upgrades were all done).

After MW upgraded to 1.44, the site was mostly working again except
for templates.

I first tried re-importing the latest templates from Wikipedia using
<https://en.wikipedia.org/wiki/Special:Export> to retrieve the latest
versions. This seemed to fix a few of the templates, and I saw stuff
in Special:RecentChanges. However, some templates like cite web just
don't seem to have been imported, even though I they were in the XML
file. And even more weirdly, when I tried to edit the `Template:Cite web`
page on TW to try to copy the Wikipedia cite web template code, MediaWiki
said the page has changed since I started editing, so I couldn't
save my manual edit. After a bunch more fussing around, I realized that
what seemed to have happened is that since the templates were created
by accounts that were _not_ on TW, the MW upgrade process seemed to
have gotten confused and just deleted the _revisions_ associated
with those pages. However, the upgrade process did not delete the actual
_content_ of the pages, so those pages were still in the `page` table
in the database. This inconsistent state, where the revisions were
gone but the pages were there, seemed to confuse MW. So what I ended
up doing is to delete the pages from the database using stuff like:

```sql
select page_id from page where page_namespace = 10 and page_title = 'Cite_web';
# suppose that printed '3346'; then I would type:
delete from page where page_id = 3346;
```

After deleting the page from the `page` table, now there is no trace
of that template page on the wiki. At this point, I re-imported the
page from the XML, and it worked.

After that, most templates were working, but cite web was still
not working. It was failing with some sort of Lua error.
I had to make the change listed [here](#cite-web-template-fix)
in order to get it working again.

Now the reference tooltips were not working. It turned out that the
reference tooltips were now 'gadgets' rather than just JS/CSS that you
could copy to your wiki, so you have to register them officially as a
'gadget'. To fix this, I had to follow the instructions
[here](https://www.mediawiki.org/wiki/Reference_Tooltips#About),
namely:

1. Make sure https://en.wikipedia.org/wiki/MediaWiki:Gadget-ReferenceTooltips.js is the latest version from Wikipedia.
2. Make sure https://en.wikipedia.org/wiki/MediaWiki:Gadget-ReferenceTooltips.css is the latest version from Wikipedia.
3. Edit the page `MediaWiki:Gadgets-definition` on TW to add the line:

   ```
   * ReferenceTooltips[ResourceLoader|default|type=general|dependencies=mediawiki.cookie,jquery.client]|ReferenceTooltips.js|ReferenceTooltips.css
   ```

4. Make sure https://en.wikipedia.org/wiki/MediaWiki:Gadget-ReferenceTooltips is the latest version from Wikipedia.

5. Make sure the Gadgets extension is loaded in LocalSettings.php. The extension should be installed by default, but not loaded, so you need to add the line:

   ```php
   wfLoadExtension( 'Gadgets' );
   ```

Now if you go to your preferences on the wiki, you should see a new
Gadets tab with Reference Tooltips as the only listed gadget.

After this, the reference tooltips were working again. You might need to
append `?action=purge` to the page and/or Ctrl-Shift-r in the browser to
make sure all caches are cleared.

A few other templates at this point were still missing (they weren't in
the weird state that e.g. cite web was in where I couldn't even edit the
page; instead, the page simply didn't exist). So for these templates,
I just had to re-export the XML from Wikipedia and re-import them.

At this point, the server ran out of disk space when doing a nightly
backup... I think before the
upgrades, the server was barely managing with just enough disk space
for the daily backups to be made and then uploaded to AWS. But the
upgrades themselves took up a bunch of space, so now there was not
enough space to do the daily backups. After discussing with Vipul,
I ended up upgrading to the next tier up in Linode.

I also deleted some old kernels that Ubuntu keeps in some weird place,
and also did apt autoremove/apt autoclean.

Once there was enough disk space and awscli was installed, the backups
worked.

At this point, the only problem was that the site was ~10x slower
than it was before the upgrade.

It turned out that part of the slowness was due to some rate-limiting
nginx logic that I had put in prior to the whole upgrades process. I'm
not entirely sure what was going on, but it seems possible to me that
the rate limiting logic was in place, but didn't really work on the
old nginx (on Ubuntu 18.04), and then once the Ubuntu got upgraded,
the new nginx started actually implementing the rate limiting logic (or
it started implementing the logic in a different or 'more effective' way).
The telltale sign was that when loading any page with images, each
image would take an extra ~1 second to load. After tweaking the
rate-limiting logic, the only slowness left was pages with tons of
references (such as timeline of SpaceX).

For the pages with tons of references, I decided to switch from the
default luastandalone to the faster luasandbox. This required
installing a package:

```bash
sudo apt install php-luasandbox
```

and then swapping it in LocalSettings.php:

```php
$wgScribuntoDefaultEngine = 'luasandbox';

# I also increased the cpuLimit because sometimes the references
# would only partially get generated, with the rest giving a
# Lua code 24 or whatever.
$wgScribuntoEngineConf['luasandbox']['cpuLimit'] = 20;
```

After this, LuaSandbox should show up in Special:Version.

I also changed one setting to enable file caching:

```php
$wgUseFileCache = true;
```

Finally, in LocalSettings I changed one spot to use `https://` instead
of `http://`.

After all this work, the site was finally loading in a reasonable amount
of time (although perhaps still a bit slower than pre-upgrade).
