---
layout: default
---

## What is Mantle?

Mantle is a programamble load balancer for POSIX metadata.

## What is the motivation for Mantle?

Ceph decouples metadata IO and data IO into separate clusters so that these
services can scale independently. Metadata workloads are susceptible to
hotspots and flash crowds of clients that access different parts of the
namespace.

## News

- 10/26/16 - Mantle was merged into Ceph! [[link](https://github.com/ceph/ceph/pull/10887)]
- 11/15/15 - Mantle was published at Supercomputing! [[link](http://dl.acm.org/citation.cfm?id=2807607)]

## Publications

1. M. Sevilla et al., *Mantle: A Programmable Metadata Load Balancer*
