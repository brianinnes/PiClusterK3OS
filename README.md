# Home Kubernetes cluster

**This is a work in progress, documenting here as I make progress**

This project aims to provide a set of instructions to setup a home Kubernetes cluster based on Rancher K3OS.

When working on kubernetes projects having a good CI/CD pipeline, which is hosted on Kubernetes is a not essential, but greatly helps.  This project will show how to create a setup which is primarily based on a Raspberry Pi cluster.

Unfortunately not all the software needed have multi-architecture container images available, or if they do they don't support images for Raspberry pi (running Raspbian - amr architecture), so I need to included at least 1 amd64 architecture cluster node, which will be used to run containers initially only available on the Intel platform.  This cluster node will also be used to do multi-architecture builds.  For the amd64 node I will use a old laptop I have.  

It is possible to use the K3OS iso image to boot and install K3OS to an old MacBook pro laptop.  I may replace this with a smaller single board Intel/AMD system in the future.

I also want to be able to run the cluster in a way it can be mode portable, so I can take an instance of the cluster and run it at different locations, hacks, meetups, etc...  To facilitate this I plan to use a system with dual ethernet ports. (I already have an [up<sup>2</sup>](https://up-board.org/upsquared) board for this), but this feature is not essential.

## Aims of the project

This project is to create a Kubernetes cluster at home using low cost hardware and open source software.  The cluster will be used to run various workloads for house automation and personal projects.

I also want the cluster to host a DevOps environment to provide build and deploy automation for personal projects.

The software stack should have the following components:

- Core Kubernetes system
- Kubernetes dashboard
- [TLS certificate management](/docs/features/TLScerts.md)
- Automatic DNS resolution for deployed workloads
- Load Balancer
- [Ingress](docs/features/upgradeTraefik.md)
- [Cluster storage](docs/features/storage.md) (for Raspberry pis, this needs to be off the micro-SD card)
- Knative / Istio

For the development environment the following components:

- local git server
- local registry for containers (and helm charts?)
- CI/CD pipeline tools

## Cluster setup

Before you start you should verify you have the [necessary requirements](docs/setup/setupRequirements.md), both hardware and software.

To setup the cluster follow the instructions on each of the following pages:

- [Raspberry Pi K3OS master and agent - arm32](docs/setup/setupArm32RaspberryPiK3OS.md)
- *optional* - [Raspberry Pi K3OS agent - arm64](docs/setup/setupArm64RaspberryPiK3OSAgent.md)
- [Intel based K3OS agent](docs/setup/setupIntelK3OSAgent.md)
- [Load balancer setup](docs/setup/setupMetalLB.md)
- [Dashboard setup](docs/setup/setupK8sDashboard.md)
- [cluster storage](docs/setup/setupClusterStorage.md)
