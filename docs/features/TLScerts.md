# TLS Certificate Management

We need to provide a way to automatically create and manage TLS cert needed within the cluster.  Kubernetes now has the Cert Manager feature, but you need an issuer to actually generate the certificates.

LetsEncrypt is a publicly, free option, but they are product [bolder](https://github.com/letsencrypt/boulder) and [pebble](https://github.com/letsencrypt/pebble) to allow you to host your own certificate authority

Need to investigate best option between bolder and pebble and verify they can work with Cert Manager.

Note:  amd64 architecture container images available on dockerhub - no arm or arm64, so if going down this route will need to generate arm images for Raspberry Pis.  As the apps are written in go, shouldn't be too difficult to do (I hope)