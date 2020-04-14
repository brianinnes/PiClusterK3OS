# Buildah on Kubernetes

```bash
dnf install -y git
git clone https://github.com/binnes/Node-RED-Docker.git
cd Node-RED-Docker
buildah bud --isolation chroot -t nodered .
```
