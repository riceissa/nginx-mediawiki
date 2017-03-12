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
I have no idea what this is, but one remedy is to export both with this box checked and with this
box unchecked.
You can always run `php rebuildrecentchanges.php` in the `maintenance` directory to
deduplicate revisions, so I'm pretty sure importing twice is fine.
(Note: I began by trying to import the XML dump with the "full history" so I have
no idea what happens if you only try to get the most recent version.)

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
```
