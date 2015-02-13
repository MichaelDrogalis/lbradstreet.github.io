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

## Operation

A user viewing the dashboard can select from a list of deployment IDs, each of
which correspond to a log for an Onyx cluster/deployment. When a deployment ID
is selected, the dashboard server replays the log. As it reads the log
messages, it applies the log entry to its current replica of the cluster state.
It uses these lower level log messages and the current replica in order to send
higher level messages to the dashboard, e.g. a job was started, a task is
running, a job is killed. These higher level messages, along with the raw log
message, are sent to the dashboard via a websocket. As the replica is replayed
the dashboard view updates until it is consistent with the current state of the
cluster.

## Loading Deployment data / replaying the log

![Loading Deployment](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/streaming_log.gif)

## Dashboard Status

![Dashboard Status](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/dashboard_status.png)

## Jobs Selector

![Jobs](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/jobs.png)

## Job Status

![Job Status](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/job_status.png)

## Running Tasks and Peers

![Running Tasks](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/running_tasks.png)

## Catalog

Something about how one of the beauties of Onyx, is that it is extremely flexible at run time.
Something about catalog and workflow here. Can view and also has the
information to manage the jobs etc.

![Catalog](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/catalog.png)

## Workflow

![Workflow](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/workflow.png)

## Log Entries Filtered by Selected Job

![Job Activity](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/job_activity.png)
