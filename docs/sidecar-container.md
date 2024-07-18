# Sidecar Container

Sidecar containers are the secondary containers that run along with the main application container within the same Pod. These containers are used to enhance or to extend the functionality of the primary app container by providing additional services, or functionality such as logging, monitoring, security, or data synchronization, without directly altering the primary application code.

Typically, you only have one app container in a Pod. For example, if you have a web application that requires a local webserver, the local webserver is a sidecar and the web application itself is the app container.

## Sidecar containers in Kubernetes
Kubernetes implements sidecar containers as a special case of init containers; sidecar containers remain running after Pod startup. This document uses the term regular init containers to clearly refer to containers that only run during Pod startup.

Provided that your cluster has the SidecarContainers feature gate enabled (the feature is active by default since Kubernetes v1.29), you can specify a restartPolicy for containers listed in a Pod's initContainers field. These restartable sidecar containers are independent from other init containers and from the main application container(s) within the same pod. These can be started, stopped, or restarted without effecting the main application container and other init containers.

You can also run a Pod with multiple containers that are not marked as init or sidecar containers. This is appropriate if the containers within the Pod are required for the Pod to work overall, but you don't need to control which containers start or stop first. You could also do this if you need to support older versions of Kubernetes that don't support a container-level restartPolicy field.

## Differences from application containers
Sidecar containers run alongside app containers in the same pod. However, they do not execute the primary application logic; instead, they provide supporting functionality to the main application.

Sidecar containers have their own independent lifecycles. They can be started, stopped, and restarted independently of app containers. This means you can update, scale, or maintain sidecar containers without affecting the primary application.

Sidecar containers share the same network and storage namespaces with the primary container. This co-location allows them to interact closely and share resources.

## Differences from init containers
Sidecar containers work alongside the main container, extending its functionality and providing additional services.

Sidecar containers run concurrently with the main application container. They are active throughout the lifecycle of the pod and can be started and stopped independently of the main container. Unlike init containers, sidecar containers support probes to control their lifecycle.

Sidecar containers can interact directly with the main application containers, because like init containers they always share the same network, and can optionally also share volumes (filesystems).

Init containers stop before the main containers start up, so init containers cannot exchange messages with the app container in a Pod. Any data passing is one-way (for example, an init container can put information inside an emptyDir volume).

## Resource sharing within containers
Given the order of execution for init, sidecar and app containers, the following rules for resource usage apply:

    - The highest of any particular resource request or limit defined on all init containers is the effective init request/limit. If any resource has no resource limit specified this is considered as the highest limit.

    - The Pod's effective request/limit for a resource is the sum of pod overhead and the higher of:
        - the sum of all non-init containers(app and sidecar containers) request/limit for a resource
        - the effective init request/limit for a resource
    - Scheduling is done based on effective requests/limits, which means init containers can reserve resources for initialization that are not used during the life of the Pod.
    - The QoS (quality of service) tier of the Pod's effective QoS tier is the QoS tier for all init, sidecar and app containers alike.
    - Quota and limits are applied based on the effective Pod request and limit.

## Sidecar containers and Linux cgroups
On Linux, resource allocations for Pod level control groups (cgroups) are based on the effective Pod request and limit, the same as the scheduler.

