---
title: Docker Repo Cluster
---

Ok, I've wanted to be able to learn about and use "microservices" for a while. Nowadays it seems like the way to orchestrate containers is using Kubernetes.  So I decided to learn by creating my blog using micro services running on a public cloud.  Great!

However, the first thing that one needs for microservices is to be able to create, store, find, and use _containers_.  I love version control, and I make many many mistakes over time, hence I want all of the state for my microservices to be store in git.  This leads me to the following subgoal: create a cluster that acts as a docker container repository.

## Requirements

- Create a docker container from a git commit
- Act as a docker repository so that I can use it to get containers for my Kubernetes microservices blog (the ultimate, original goal)
- be high-availability
- be tied into continuous-integration and continuous deployment scenarios
  - so automatically create images from tagged commits, say
- be fully scripted and scalable if need be


## Docker Registry

It turns out that running a docker registry is "as simple" as running a particular docker container [https://docs.docker.com/registry/deploying/].  Even better, the state of the registry (the known images) can be configured to us S3 (or Google Cloud buckets, or OpenStack).  So first let's try to get a working HA docker registry.  I will be creating this registry inside of a VPC so I will eschew almost all security concerns.  We will have 2 instances of the docker registry running, and 2 instances of nginx round-robing proxy to these registries. We will use `Keepalived` in active-active setup to ensure that a single elastic IP is routed to a running nginx (between the 2 instances):

- start 2 instances running docker registry container configured for S3 storage
- start 2 instances running nginx both proxy to both docker instances
- start keepalived on the nginx instances in an active-active setup for failover

### VPC and instances

Ok, we'll want to create the following:
- 1 VPC
- 4 instances (micro) for the docker registry
- 1 "bastion" instance to SSH into the VPC
- 1 elastic ip (EIP) attached to the bastion

We'll use Terraform and Packer from Hashicorp to create the architecture and the base images for the instances on AWS.  We'll also use jplankton.project-manager.shunt as a template engine for the various files needed for this infrastructure.  The instances will be provisioned with SaltStack.  See the post about [Bastioned VPC] which details the general architecture of running a bastion + VPC with a number on instances inside provisioned using SaltStack.

The bastion and VPC are the same as the post above, however we need to add the provisioning for Keepalived, Docker and Nginx using salt.
