# CI / CD Pipeline

Part of the reasone for building this cluster is to experiment and learn about building on cluster using the latest available tooling, such as Tekton and possibly Jenkins X.

Like many project initiated by Google, Tekton is currently limited to the **amd64** architecture and the entire project is not well architected or documented to easily make it multi-architecture enabled, so initially it will have to be restricted to only use **amd64** nodes.  This is possible by adding nodeSelectors to taskRun and pipelineRun configurations.

Ideally tasks should be able to be run on nodes of any supported architecture, but further investigation will need to be done.  I imagine this work will be a 2 part project:

- get task pods to run on all architectures
- get the underlying Tekton deployment to run on all architectures

Another investigation needed is to see how Tekton can be used to create multi-architecture (fat manifest) images from source (git repo)

## Jenkins X

Jenkins X is a tool built on Tekton pipelines.  Ideally this should be the tool of choice rather than the lower level Tekton pipelines.  However, Jenkins X currently requires backlevel Helm (v2), which then requires the *Tiller* cluster side deployment and I do not want to downgrade Helm to support Jenkins X, so until the Jankins X project can resolve the backlevel dependency Tekton will be used directly.