---
layout: page
---

## Papers I've read

### 2016-08-16

(2003) [Google File System](http://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)
- I was looking to see how they handled placement and reads. The paper didn't provide too many details other than that racks are a factor in replica placement.

(2014) [HDFS for geographically distributed filesystem](http://cdn.oreillystatic.com/en/assets/1/event/120/HDFS%20for%20Geographically%20Distributed%20File%20System%20Presentation.pdf) ([talk](http://conferences.oreilly.com/strata/strataeu2014/public/schedule/detail/39174))
- A future improvement is to consider network proximity (slide 22), but there's also a GeoNode that appears to do some of this (slide 25).
- Something I hadn't considered is how jurisdiction plays in. For example, Saudi Arabia required data to stay within its borders (slide 36).

(2012) [CFS vs HDFS](http://www.datastax.com/wp-content/uploads/2012/09/WP-DataStax-HDFSvsCFS.pdf)
- CFS sounds pretty cool, like it does geo distribution. It'd be nice to see how they choose locations for replicas.
- [Deploying Cassandra across multiple datacenters](http://www.datastax.com/dev/blog/deploying-cassandra-across-multiple-data-centers) and [Multi-DC](http://docs.datastax.com/en/archived/cassandra/0.7/docs/operations/datacenter.html) have some info. Strategy info provides a target replication factor per datacenter. A "snitch" figures out where actual hosts are w.r.t. racks and dcs.
- [`sortByProximity`](https://github.com/apache/cassandra/blob/3dcbe90e02440e6ee534f643c7603d50ca08482b/src/java/org/apache/cassandra/locator/AbstractEndpointSnitch.java#L42-L56) considers DC and rack

### 2016-08-13

(2000) [piranha](http://barroso.org/publications/isca00.pdf) h/t [@loveapaper](https://twitter.com/loveapaper/status/764510974183432192) - describes a scalable parallel CPU architecture. I found the communication protocol interesting.

(1969) [lisp garbage collector](https://www.cs.purdue.edu/homes/hosking/690M/p611-fenichel.pdf) also h/t [@loveapaper](https://twitter.com/loveapaper/status/764405276434923520) - in the new world of virtual memory, while there's basically infinite space to store **billions** of values.
