# Registry

For the development we want a local registry installed on the cluster.  Plan is to use the Docker Registry, which is provided in a container on [dockerhub](https://hub.docker.com/_/registry)

## Pushing from docker buildx into registry

The registry is exposed via the kubernetes ingress, so has a self-signed certificate.  To allow the docker builder to push to the registry it needs to be told to ignore the use of the self-signed certificate (which it cannot verify using it well known CA root certificates).

To push to the registry replace **--push** with **--output type=image,registry.insecure=true,name=HOST/USER/IMAGE_NAME,push=true**, replacing HOST, USER and IMAGE_NAME with appropriate values:

- Host : hostname of the registry (registry.bik8s.home)
- USER : username of the registry user (your own username)
- IMAGE_NAME : name of the image

## TODO

Investigate the following:

- Registry viewer **hyper/docker-registry-web** image
