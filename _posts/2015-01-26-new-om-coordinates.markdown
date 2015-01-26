---
layout: post
title:  "Public Service Announcement: Om's new coordinates"
date:   2015-01-26 22:19:05
categories: clojurescript
---

Om's leiningen coordinates have been changed from \[om "version"\] to
\[org.omcljs/om "version"\] (with a brief stopover at \[org.om/om "version\] -
alas the domain om.org was already taken). If you've noticed that lein ancient
(https://github.com/xsc/lein-ancient) hasn't picked up the latest versions,
this is why.

The change in the coordinates has another, more important, consequence: you
will need to exclude om from any dependency that depends on the old coordinates
or you will get some weird behaviour (e.g. inability to use the great new
reference cursor implementation!).
