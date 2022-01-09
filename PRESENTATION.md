---
title: Highly Available Elasticsearch
description: Presentation
marp: true
paginate: true
theme: gaia
backgroundColor: white
# class:
# - lead
# - invert
---
<!--
_class: lead
_paginate: false
-->
<style>
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@300;400;500&display=swap');
* {
    font-family: 'Open Sans', sans-serif;
}
</style>

![logo](./assets/logo.png)

# Mustafa Can Sevinc

*10.01.2022*

---

## 💼 My Elasticsearch Experience

<style scoped>p, ul, li, strong { font-size: 40px; }</style>

* Since 6.2.4 to 7.2

* Various ES clusters like:
    * One-node clusters
    * Two-node clusters
    * Three-node clusters
    * Using Ceph RBD as massive storage in data roles
    * To sync metadata of objects in s3 buckets 
---

## 📝 My Submission

<style scoped>p, ul, li, strong { font-size: 38px; }</style>

I've done the first time while my assignment submission:

* Configure roles in a single nodeSet
* max_map_count using an initContainer instead of `node.store.allow_mmap: false`

Different on new elasticsearch versions:

* Optimized auto-configuring most of the settings
* The License

---

### 🎯 Features

<style scoped>p, ul, li, strong { font-size: 40px; }</style>


* ☁️ Works on Kubernetes

* 🌀 High availability

* 🎭 Identical roles

* ✍️ README-driven

* 🧩 Kustomize-generated resources

* 🪄 Easily applicable

---

### 📐 Solution Architecture

* 💪 Resilience
* 🎭 Roles
* 💎 Sharding
* 💾 Storage
* 🧮 Memory & JVM Size
* 🥽 Virtual Memory
* 🛠️ Applying Custom Configuration
* 📈 Benchmark

---

### 📐 Solution Architecture

<style scoped>p, ul, li, strong { font-size: 32px; }</style>

**💪 Resilience**

* Resilient if:
    * green,
    * at least two data nodes,
    * at least one replica for each shard,
    * at least thee master nodes,
    * load balancer
* Taking regular snapshots: SLM
* Design: Identical three nodes to ensure resilience to single-failure-node

---

#### 📐 Solution Architecture

<style scoped>p, ul, li, strong { font-size: 38px; }</style>

**🎭 Roles**

* master

* data

* ingest

* ml

---

### 📐 Solution Architecture

**💎 Sharding**

Aim for:
* Shards between 10GB and 50GB.
* Max 20 shards per GB of heap memory

Avoid:
* Unnecessary mapped fields by using explicit mapping

---

### 📐 Solution Architecture

<style scoped>p, ul, li, strong { font-size: 32px; }</style>

**💾 Storage**
* Network-attached PersistentVolumes
* Local PersistentVolumes

**🧮 Memory & JVM Size**

Xms and Xmx should be
* Same with each other
* Set to no more than 50% of the total available RAM
* Less than 26GB

---

### 📐 Solution Architecture

<style scoped>p, ul, li, strong { font-size: 30px; }</style>

**🥽 Virtual Memory**
* Elasticsearch uses memory mapping.
* `vm.max_map_count` should be set to `262144`

**🛠️ Applying Custom Configuration**

* Create a custom image
* Use init containers

**📈 Benchmark**

* Rally can be used to size the cluster correctly

---

<style>
footer {
    text-align: center;
}
</style>

![bg 62%](./assets/diagram.png)

<!--
_footer: The Diagram of The Solution
-->

---

### 💡 The Solution - Configuration

<style scoped>p, ul, li, strong { font-size: 38px; }</style>

* ECK with **vanilla manifest files**
* Elasticsearch and Kibana with **kustomize**-generated file
* Configured using **initContainers**
* LoadBalancer
* Dynamic mapping option
* AWSElasticBlockStore
* Master & Data roles
* SLM Policy

---

### 💡 The Solution - Defaults

<style scoped>p, ul, li, strong { font-size: 38px; }</style>

* Total shards per node
* JVM Heap Size Settings
* Update strategy
* PodDisruptionBudget configuration
* Node scheduling
* Readiness probe configuration
* PreStop hook configuration
* Security context configuration

---

### 🚀 Deployment

<style scoped>p, ul, li, strong { font-size: 40px; }</style>

1. Install ECK Custom Resources

2. Install ECK Operator

3. Monitor the operator logs

4. Generate elasticsearch & kibana resources

5. Deploy elasticsearch & kibana

6. Verify everything is ready-to-use

---

<!--
_class: lead
_paginate: false
-->

# ❇️

# Thanks for your time