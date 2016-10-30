---
layout: default
---

## What is the difference between Mantle and the load balancers in CephFS?

## How does CephFS balance metadata load?

## Why did we design Mantle?

While the numbers in [2] clearly show an improvement, quantifying the
components that lead to the performance gains is challenge. What is the
metadata cluster **really** doing? Which directory inodes are moving? When are
they moving? Why are the load balancing decisions being made? Are the decisions
optimal?

## How does Mantle balance metadata load?

## Publications

1. M. Sevilla et al., *Mantle: A Programmable Metadata Load Balancer for the
Ceph File System*, SC 2015,
[[link](http://dl.acm.org/citation.cfm?id=2807607)]

2. P.  Donnelly, *Initial results of CephFS performance evaluation*, Ceph
Mailing List, [[link](http://marc.info/?l=ceph-devel&m=147494693010640&w=2)]
