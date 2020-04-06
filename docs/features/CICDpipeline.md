# CI / CD Pipeline

Part of the reasone for building this cluster is to experiment and learn about building on cluster using the latest available tooling, such as Tekton and possibly Jenkins X.

Like many project initiated by Google, Tekton is currently limited to the **amd64** architecture and the entire project is not well architected or documented to easily make it multi-architecture enabled, so initially it will have to be restricted to only use **amd64** nodes.  This is possible by adding nodeSelectors to taskRun and pipelineRun configurations.

Ideally tasks should be able to be run on nodes of any supported architecture, but further investigation will need to be done.
