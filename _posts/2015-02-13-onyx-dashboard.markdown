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
Onyx is implemented using a masterless design, i.e. there is no single process
or entity that coordinates the peers. Rather, peers in Onyx coordinate via a
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
have leveraged this information in order to create a dashboard that gives a
high level overview of the state of an Onyx cluster.

## Operation

Open opening the dashboard, users are provided with a dropdown containing
deployment IDs, each of which correspond to a log for an Onyx
cluster/deployment. When a deployment ID is selected, the dashboard server
replays the log. As it reads the log messages, it applies the log entry to its
current replica of the cluster state.

The dashboard interprets these lower level log messages, and the current
replica, in order to send higher level messages to the dashboard e.g. a job was
started, a task is running, a job is killed. These higher level messages and
the raw log entry are sent to the dashboard client via a websocket. As the
replica is replayed the dashboard view updates until it is consistent with the
current state of the cluster.

## Dashboard Selection & Log Replay

![Loading Deployment](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/streaming_log.gif)

The above animation demonstrates the selection of an Onyx deployment, and the log replay.

## Dashboard Status

![Dashboard Status](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/dashboard_status.png)
![Dashboard Status](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/loading_deployment.gif)

The Dashboard Status section shows the time of the last intepreted log entry.
As log messages are replayed, the status of the dashboard will advance until it
is up to date with the current state of the cluster. When the dashboard is up
to date with the cluster replica, the panel turns grey to signal that it has
fully replayed the log.

## Deployment Peers

![Deployment Peers](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/deployment_peers.png)

Displays the cluster peers that are running for the deployment. An additional
heartbeat check is performed by the dashboard in order to ensure that the peers
described by the log are still alive.

## Deployment Log Entries

Show deployment log entries here? Will be showing the filtered log later but maybe it's good to see them here as it's a bit of perspective about how the peers coordinate?

## Jobs Selector

![Jobs](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/jobs.png)

This section section displays the jobs that have been started for a replica,
sorted by their start date. Clicking on a job will enable additional job
related panels that show information about the job in question, and also
filters the raw log entries to only show log entries relating to that job.

## Jobs Management

![Jobs](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/job_management.png)

The Job Management section provides functionality to manage jobs. This allows
users to kill jobs by writing a "kill job" entry to the log which will be read
by the peers. It also allows for jobs to restarted by killing the job, assuming
it is still running, and then start a new job using the same workflow and
catalog data as the job.

## Job Status

![Job Status](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/job_status.png)

The Job Status section shows the current status of the job (pending, running,
incomplete, finished, and killed), and the task scheduler used (currently round
robin and percentage are available).

## Running Tasks and Peers

![Running Tasks](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/running_tasks.png)

The Running Tasks section shows the workflow tasks that are being processed by
the cluster, as well as the peers that are processing them. In the above
example, `:read-rows` is being processed by multiple peers.


## Workflow

![Workflow](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/workflow.png)

The Workflow section shows the workflow for the selected job. Workflows
describe the DAG dataflow graph for the job's computation. One of the key
benefits to the design of Onyx is that the workflow design is flexible and is
pure data, submitted when running a job, resulting in the data being serialised
to the log where it is then read by the peers.  The dashboard leverages this to
read the DAG structure for the job and display it for dashboard users.

## Catalog

![Catalog](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/catalog.png)

Jobs under Onyx must also be submitted with a catalog, which describes the
tasks included in the corresponding workflow for the job. Information contained
in the catalog might include the function run for a task, some parameters
supplied to the function that is run, or possibly some information required to
initialise the task e.g. the sql provider that should be used by an SQL input
plugin.  

## Log Entries Filtered by Selected Job

![Job Activity](https://raw.githubusercontent.com/lbradstreet/lbradstreet.github.io/dashboard-post/images/dashboard/job_activity.png)

The Log Entries component shows the raw log entries for the deployment, or for
the job that is selected. These allow users to see the raw activity for the cluster

## Future Work

As you have seen, we've created a basic dashboard that may be useful for some
monitoring or debugging use cases. 

There is more that could be done in the future. As Onyx jobs are described as
data, there is the potential to allow for client side editing, manipulation,
and submission of jobs. In addition, debug mode jobs could be run, which would
place additional nodes in between each workflow node, which could stream the
segment results to the dashboard client. 

In addition, we are looking into best practices for monitoring cluster metrics.
While this data would not be written to the cluster log, we could provide
integration between dashboard and metrics visualisation tools. The combination
of log interpretation with cluster metrics would substantially increase the
power of these metrics, as information about the cluster (peer joining or
leaving, scheduler used, etc) can be combined with throughput and latency
metrics.

If you are interested in Onyx, [take a
look](https://github.com/MichaelDrogalis/onyx) and try out the [Onyx starter
tutorial](https://github.com/MichaelDrogalis/onyx-starter), which you can use
the newly minted dashboard with [FIXME instructions here](FIXME).
