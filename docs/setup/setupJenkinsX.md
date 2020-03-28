# Setup Jenkins X

This will also install Tekton

## Install the client CLI

```brew install jenkins-x/jx/jx```

For other options see [here](https://jenkins-x.io/docs/getting-started/setup/install/)

## Install Jenkins on cluster

Uninstall helm 3 on your workstation if it is installed and install helm 2 with command (for mac) ```brew install helm@2``` followed with command ```ln -s /usr/local/Cellar/helm@2/2.16.3/bin/helm /usr/local/bin/helm``` then run the following command (follow the prompts and accept the defaults)

```jx install --provider=kubernetes --on-premise```

Select **Serverless Jenkins X Pipelines with Tekton**, then follow the prompts

## Options to investigate

- intel only?  May need to update to target only amd64 arch
- helm 2 isn't a great option, can we get round this? Is it an install only issue, or does it need to be installed for jenkins X operation?
