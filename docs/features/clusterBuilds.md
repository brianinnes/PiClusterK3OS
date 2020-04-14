# Building Containers on a Kubernetes cluster

The most common tool developers use to create a container has been the Docker CLI.  However, there are now a number of tools available to create containers.

When choosing which tool to adopt there are a number of features to consider:

- support for manifest list for multi-architecture support
- ability to work within a Kubernetes cluster
- support for container runtimes
- function / features of the tool

## Tools and multi-architecture support

To be able to build multi-architecture images the tooling needs to be able to create a manifest list, which combines different architecture images into a single registry entry allowing systems of different architectures to pull a single image and each system will receive the appropriate image which matches the architecture of the system pulling the image.  Not all tooling supports manifest lists:

|Tool|Support|
|----|-------|
| [Docker](https://www.docker.com/) | Supported with buildx and manifest sub-commands|
| [Kaniko](https://github.com/GoogleCloudPlatform/kaniko) | No support - https://github.com/GoogleContainerTools/kaniko/issues/1102|
| [Podman](https://github.com/containers/libpod) | ? |
| [Buildah](https://github.com/containers/buildah) | Supported with the manifest sub-command |
| [BuildKit](https://github.com/moby/buildkit) | ? - should be OK as this is what Docker uses |
| [img](https://github.com/genuinetools/img) | Supported as runs underlying BuildKit - also claims to work on Kubernetes |

