Kubernetes allow user to declaratively manage the desire state of system. But much like any other declaritive state management system sometime imparitive and itempotent is needed to amplify its power and provide ordering and provide powerful templitization functionality.

The tools in the collection allows user of ACM to be able to connenect to managed kubernetes cluster and do ANYTHING on the managed cluster through Ansible or scripts.

With the power of Ansible these actions don't have to be limited to the managing Kubernetes resources. We can expend to anything related to the cluster that Ansible can manage and cooredinate in cluster action (things that Kubernetes have control over) with out of the cluster action (things that Kubernetes don't have control over, like infrastructure configuration).

