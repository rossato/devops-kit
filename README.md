# DevOps Kit

This repository is a collection of Ansible, ArgoCD, Kustomize, and Helm resources to demonstrate an end-to-end Kubernetes cluster automation workflow.  I developed this collection with Red Hat OpenShift in mind, but with small modifications it can be adapted to other Kubernetes environments.

## Organization

The top-level directories follow a rough chain of dependencies.  Each folder could be stored in its own repository (or multiple repositories, in the case of apps).

### 1-bootstrap

This contains an Ansible playbook to initialize a newly-installed cluster.  It contains the minimum necessary to bootstrap an ArgoCD instance:

* Installation of `ImageContentSourcePolicies` and `CatalogSources` for a disconnected cluster
* Installation of the Red Hat `openshift-gitops` operator, including installation of the default ArgoCD instance
* Installation of a root infrastructure "App of apps"

The openshift_common and openshift_gitops roles are modifications of the excellent work done by Ales Nosek and contributors at https://github.com/noseka1/ansible-base.  These roles are offered under their original Apache 2.0 license (license text in the 1-bootstrap folder).

### 2-infra

The infrastructure "app of apps" points to a directory with the cluster name under the `clusters/` directory.  This is expected to point to other resources in the `2-infra` repo that are needed to prepare a cluster for users:

* One or more cluster `archetypes` that contain resources that all clusters of that type will need
* A `common` directory with implementations of cross-cutting concerns (e.g. auth)
* An `apps` directory that contains IT-managed application resources (groups, RBAC, namespaces, and quotas)

The `common/app-gitops` directory contains a second instance of ArgoCD that will allow application teams to directly manage their own resources.  This instance is cluster-scoped but manages a restricted list of resource types.  In particular it does not manage namespaces, so it cannot give itself permissions to edit projects that the "infra" Argo instance hasn't assigned to it.

The "app" Argo instance uses `AppProjects` to isolate app teams from each other.  If the `AppProject` allows the app-of-apps pattern, it cannot prevent `Applications` from escaping their project, but `sourceRepo` rules can prevent apps from attaching themselves to projects where they do not belong.

### 3-apps

This contains the dev-team-facing resources needed to manage applications.  The permission model should allow these resources to be edited by dev teams directly in their own repo without the need for IT-team sign-off.

* `demo`: a simple, Kustomize-based deployment
* `quarkus-getting-started`: a Helm-based deployment and Helm-deployed Tekton CI pipeline

### 4-cicd

This contains generic CI/CD Helm charts that could be managed by a central "tools" team.  Note that because app teams manage their own deployments, they can fork these resources and tailor them to their own needs.
