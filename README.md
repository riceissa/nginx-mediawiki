# nginx-mediawiki

Instructions for setting up MediaWiki on nginx on Ubuntu 16.10

Full list of software:

- Ubuntu 16.10
- nginx
- MediaWiki
- MediaWiki templates
- php7.0-fpm
- certbot (letsencrypt)
- Linode for domain stuff

## MediaWiki extensions

- Scribunto
- ParserFunctions
- Cite

## MediaWiki imports

[MediaWiki's instructions](https://www.mediawiki.org/wiki/Help:Templates#Copying_from_one_wiki_to_another)
specifically say to 'Uncheck the box "Include only the current revision, not the full history".'
**However** I find that checking this returns a very old version of the page for at least one
page (`MediaWiki:Common.css`).

```
Template:Reflist
Template:Snd
MediaWiki:Common.css
MediaWiki:Common.js
```
