<div align="center">

[![logo](logo.png)](highly-available-elasticsearch)

[![Releases](https://img.shields.io/github/v/release/mustafacansevinc/highly-available-elasticsearch?style=for-the-badge)](https://github.com/mustafacansevinc/highly-available-elasticsearch/releases) [![Licence](https://img.shields.io/github/license/mustafacansevinc/highly-available-elasticsearch?style=for-the-badge)](./LICENSE) [![Total lines](https://img.shields.io/tokei/lines/github/mustafacansevinc/highly-available-elasticsearch?style=for-the-badge)](https://github.com/mustafacansevinc/highly-available-elasticsearch/)

badgeMadeWith badgeBestPractices?

</div>

---

<div align="center">

SCREENSHOT OF SOLUTION

*Description of screenshot*

![kubernetes](https://img.shields.io/badge/kubernetes-326ce5.svg?&style=for-the-badge&logo=kubernetes&logoColor=white) ![elasticsearch](https://img.shields.io/badge/ElasticSearch-0779A1?style=for-the-badge&logo=elasticsearch&logoColor=white) ![kibana](https://img.shields.io/badge/Kibana-EF5098?style=for-the-badge&logo=Kibana&logoColor=white) ![aws](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white) ![markdown](https://img.shields.io/badge/Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white)

[Features](#-features) | [Solution Architecture](#-solution-architecture) | [Getting Started](#-getting-started) | [Configuration](#%EF%B8%8F-configuration) | [Maintenance Steps](#-maintenance-steps) | [Author](#%EF%B8%8F-author) | [Credits](#-credits) | [Failures](#-failures)

</div>

## 🎯 Features

• **Elasticsearch cluster in Kubernetes cluster:** At this project elasticsearch cluster is in the kubernetes cluster to ... It is only needed a kubeconfig file to use the remaining k8s cluster.

• **Deployed via Helm:** Helm is used to deploy the kubernetes cluster. It could be deployed using ... and ... but helm has advantage to ...

• **Highly available:** The elasticsearch cluster has high availability due to ...

• **Same roles:** The roles assigned to each of the elasticsearch nodes are all identical.

• **Downloadable:** The solution can be downloaded by cloning the project or downloading the zip file under Releases

• **Well documented:** All phases of the solution are documented and this README-driven project...

• **Easy appliable:** When all of the steps are completed successfully, one may own an elasticsearch cluster...


## 📐 Solution Architecture

Elasticsearch is a distributed, search and analytics which gives the near real-time search experience and analysis of data. It is capable of handling the volume and free-form nature of the data. It is fast, distributed, scalable and resilient.

And it is assumed that the team already has prior experience with it.

But running an elasticsearch cluster in production is not an easy task. There are GB's of data coming in all the time and need to make it easiliy accessible and resilient. As an elasticsearch cluster can be spawn quickly to use in production on AWS, the optimization of the cluster will be cruical to improve the speed and sustainability.

You can run out of memory, have too many shards, bad rolling shard policies, no index lifecycle management...

This solution will be focused on the resilience of the elasticsearch cluster and the design.

### What is Resilience?

Sometimes the cluster may experience hardware failure or a power loss. (underlying) The cluster will be resilient to the loss of *any node* as long as:

* The cluster health status is green.
* There are at least two data nodes.
* Every index that is not a searchable snapshot index has at least one replica of each shard, in addition to the primary.
* The cluster has at least three master-eligible nodes, as long as at least two of these nodes are not voting-only master-eligible nodes.
* Clients are configured to send their requests to a load balancer that balances the requests across an appropriate set of nodes.

Some features are offered by Elasticsearch to keep the cluster highly available. 

* [**Cross Cluster Replication**](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html): It lets you replicate index and the data across remote follower clusters which may be in a different datazone from the leader cluster so that search request can be still handled during the datacenter outage:
    * In uni-directional configuration, there is one cluster to contain only leader indices and the other cluster to contain only follower indices. It can be applied the architectures:
        * Single disaster recovery datacenter
        * Multiple disaster recovery datacenter
        * Chained replication
    * In bi-directional configuration, each cluster contains both leader and follower indices.
        * Bi-directional replication
    * Please note that CCR is not covered in this solution as high availability should be provided using only one cluster.

* [**Taking regular snapshots**](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html): So that you can restore a fresh copy of your data.
    * The snapshot creation and retention process could be automated using Snapshot Lifecycle Management (SLM).
    * If multiple SLM policies are defined to create snapshots at different time intervals, it lets you restore data from a wider range.

* [**Design for resilience**](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html): The main step of high availability is the design for resilience as an Elasticsearch cluster can continue operations as normal if some of its nodes or components are unavailable when there are enough well-connected nodes with a proper design.
    * One node clusters: A single node cluster *cannot be resilient* as there are no replicas. It is needed to override the `number_of_replicas` value as 0 to ensure the cluster can report a `green` status. A snapshot may be restored if the node fails. One-node clusters are not recommended to use in production.
    * Two-node clusters: A two-node cluster *cannot be resilient*. The master elections are majority-based, so one of the nodes should be configured as `node.master: false` to ensure it is *master-ineligible*, as if the both of the nodes were *master-eligible* the master election will fail. With this type of design, the cluster may tolerate the loss of the *master-ineligible* node. Also, a resilient load balancer should be configured to balance client requests across the nodes.
    * **Three-node clusters**: All three nodes can have the same roles in this scenario and as they all are *master-eligible*, the cluster will be resilient to the loss of any single node. Also, a resilient load balancer should be configured to balance client requests across the nodes.
    * Clusters with more than three nodes: It is good practice to limit the number of *master-eligible* nodes to three, which will shorten the time of master elections. But as the cluster is larger, it is recommended to use dedicated nodes for each role to scale resources for each task independently. Also, you should avoid sending any client requests to the dedicated *master-eligible* nodes to overwhelm them with unnecessary extra work that could be handled by one of the other nodes.
    * Multizone clusters: [Resilience in larger clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design-large-clusters.html)


### Sharding

In Elasticseach, the data is organized in indices, and those are each made of shards that are distributed across multiple nodes.

Each shard is a seperate [Lucene Index](https://lucene.apache.org/core/9_0_0/core/org/apache/lucene/codecs/lucene90/package-summary.html), made of little segments of files located on your disk. Whenever you write, a new segment will be created. When a certain amount of segments is reached, they are all merged. Whenever you need to query your data, each segment is searched.

So shards are created when a new document needs to be indexed, then a unique id is being generated and the destination of the shard is calculated based on this id.

Once the shard has been delegated to a specific node, each write is sent to the node.

This method allows a reasonably smooth distribution of documents across all of your shards. This will distribute your documents pretty well across all of your shards and you can easily and quickly query thousands of documents in the blink of an eye.

Distribution of the primary shards to other nodes implies that all shards can share the workload to improve the overall performance. Altough replication of the shards to other nodes ensures that you always have the data even if you lose some nodes, which is a key-goal of this project: **High availability**.


So we have to mention that again, buiding an elasticsearch cluster ready for production is not an easy task.

Yes, elasticsearch works even with an unoptimized configuration, but as there are more data, there will be more errors to encounter. It is always better to take time while building your cluster. You need to understand what your requirements are and what you want before building your cluster. And tune the configuration as you encounter errors, till you stable your cluster.



## 🚀 Getting Started

## 🛠️ Configuration

WIP

The elasticsearch cluster will be configured to ensure high availability and resilience to the loss of any single node.

## 👣 Maintenance Steps

## 🙋‍♂️ Author

Mustafa Can Sevinç

[![github](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/mustafacansevinc) [![linkedin](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/mcansevinc/) [![gmail](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:mcansevinc@gmail.com) [![cansevinc](https://img.shields.io/badge/website-667881?style=for-the-badge&logo=About.me&logoColor=white)](http://cansevinc.com.tr/)

## 📚 Credits

* https://tom.preston-werner.com/2010/08/23/readme-driven-development.html
* https://github.com/alexandresanlim/Badges4-README.md-Profile
* https://www.ashnik.com/how-to-configure-highly-available-elasticsearch/
* https://dzone.com/articles/apache-lucene-a-high-performance-and-full-featured
* https://www.cncf.io/blog/2021/03/25/how-to-build-an-elastic-search-cluster-for-production/

## 🍐 Failures