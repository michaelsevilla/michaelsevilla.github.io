---
layout: default
---

## What is Mantle?

A programmable load balancer for POSIX metadata.

## Why does POSIX metadata need a balancer?

File system metadata workloads impose small and frequent requests on the
underyling storage. This skewed workload makes it difficult to scale metadata
IO in the same way that data read/write throughput scales.  CephFS has
strategies for helping metadata IO scale, including decoupling metadata/data IO
and providing mechanisms for detecting load and migrating metadata to other
servers. 

## How does Mantle help us balance load?

Mantle provides a general framework and specification for expressing load
balancing policies on the *same storage system*. For example, `when()` is a
callback provided by the API and can be programmed to initiate balancing under
different conditions:

```lua
-- balance when my neighbor is idle
if MDSs[whoami+1]["cpu"]>0.25

-- balance when I have load and my neighbor does not
if MDSs[whoami]["load"]>.01 and
   MDSs[whoami+1]["load"]<.01 then
```

This lets us compare load balancing strategies instead of the properties
storage systems themselves and helps future administrators understand the
trade-offs of the metadata migration decisions.

## Publications

1. M. Sevilla et al., *Mantle: A Programmable Metadata Load Balancer for the
Ceph File System*, SC 2015,
[[link](http://dl.acm.org/citation.cfm?id=2807607)]

2. P.  Donnelly, *Initial results of CephFS performance evaluation*, Ceph
Mailing List, [[link](http://marc.info/?l=ceph-devel&m=147494693010640&w=2)]
