---
layout: post
title:  "Onyx Dashboard - Embrace the Log"
date:   2015-02-13 20:59:03
categories: clojure, onyx, distributed systems
---

## Onyx and Dashboard Overview

[Onyx](https://github.com/MichaelDrogalis/onyx) is a fault tolerant distributed
computation system, in the mold of Storm or Spark, which can be used for batch
and stream processing. As of
[0.5.0](http://michaeldrogalis.github.io/jekyll/update/2015/01/20/Onyx-0.5.0:-The-Cluster-as-a-Value.html),
Onyx is designed such that it is masterless, i.e. there is no single process or
entity that coordinates the peers. Rather, peers in Onyx coordinate via a
shared, immutable, log that is written to
[ZooKeeper](http://zookeeper.apache.org/).

When Onyx peers start up, they replay this shared log until they are up to date
with the current state (replica) of the cluster. They each continue to track
this log in order to make decisions about job management (starting, killing
jobs), job scheduling, and task scheduling (scheduling between tasks within a
job).

One side benefit of this design is that it gives a replayable view of changes
within the cluster over the time, as well as a way to view the overall state of
the cluster at a given time, even if the cluster has long been shutdown. We
have leveraged this information in order to produce a dashboard that gives a
high level overview of the state of an Onyx cluster.

## Architecture

Server replays the log. As it reads the log messages, it applies the log entry
to its current replica (state) of the system. It interprets these lower level
log messages and the current replica in order to send higher level messages to
the dashboard, e.g. a job was started, tasks are running

it streams the raw log entries via websockets. 
