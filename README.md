# Catalogue

This repository defines common tasks, pipelines and operations. They are
designed to be reused in many pipelines and applications.

## Tasks

There are two kinds of tasks providing different levels of scope within a
cluster. They are as follows:

1. `ClusterTask` with a cluster scope. This can be installed by a cluster
   operator and made available to users in all namespaces.
1. `Task` with a namespace scope. This can only be installed by an operator
   with access to the namespace, and only effects resources within that
   namespace.
