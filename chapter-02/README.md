# Chapter 2. Infrastructure Security
Many Kubernetes configurations are insecure by default. 
## Host hardening
This covers the choice of operating system, avoiding running nonessential processes on the hosts, and host-based firewalling.
### Choice of Operating System
Two popular examples of modern immutable Linux distributions designed for containers are Flatcar Container Linux (which was originally based on CoreOS Container Linux) and Bottlerocket (originally created and maintained by Amazon).
### Nonessential Processes
If you are using an immutable Linux distribution optimized for containers, then nonessential processes will have already been eliminated and you can only run additional processes/applications as containers.
### Host-Based Firewalling
Fortunately, some Kubernetes network plug-ins can help solve this problem for you. For example, several Kubernetes network plug-ins, like Weave Net, Kube-router, and Calico, include the ability to apply network policies. You should review these plug-ins and pick one that also supports applying network policies to the hosts themselves (rather than just to Kubernetes pods). This makes securing the hosts in the cluster significantly simpler and is largely operating system independent, including working with immutable Linux distributions.
### Always Research the Latest Best Practices
As new vulnerabilities or attack vectors are identified by the security research community, security best practices evolve over time. Many of these are well-documented online and are available for free.

For example, the Center for Internet Security maintains free PDF guides with comprehensive configuration guidance to secure many of the most common operating systems. Known as CIS Benchmarks, they are an excellent resource for making sure you are covering the many important actions required to secure your host operating system. You can find an [up-to-date list of CIS Benchmarks on their website](https://www.cisecurity.org/cis-benchmarks). Please note there are Kubernetes-specific benchmarks, and we will discuss them later in this chapter.

## Cluster hardening
This covers a range of configuration and policy settings needed to harden the control plane, including configuring TLS certificates, locking down the Kubernetes datastore, encrypting secrets at rest, credential rotation, and user authentication and access control.
### Secure the Kubernetes Datastore
### Secure the Kubernetes API Server
One layer up from the etcd datastore, the next set of crown jewels to be secured is the Kubernetes API server. As with etcd, this can be done using x509 PKI and TLS. The details of how to bootstrap a cluster in this way vary depending on the Kubernetes installation method you are using, but most methods include steps that create the required keys and certificates and distribute them to the other Kubernetes cluster components. It’s worth noting that some installation methods may enable insecure local ports for some components, so it is important to familiarize yourself with the settings of each component to identify potential unsecured traffic so you can take appropriate action to secure them.

### Encrypt Kubernetes Secrets at Rest
### Rotate Credentials Frequently
### Authentication and RBAC
### Restricting Cloud Metadata API Access
### Enable Auditing
Kubernetes auditing provides a log of all actions within the cluster with configurable scope and levels of detail. Enabling Kubernetes audit logging and archiving audit logs on a secure service is recommended as an important source of forensic details in the event of needing to analyze a security breach.

The forensic review of the audit log can help answer questions such as:
  * What happened, when, and where in the cluster?
  * Who or what initiated it and from where?

In addition, Kubernetes audit logs can be actively monitored to alert on suspicious activity using your own custom tooling or third-party solutions. There are many enterprise products that you can use to monitor Kubernetes audit logs and generate alerts based on configurable match criteria.
The details of what events are captured in Kubernetes audit logs are controlled using policy. The policy determines which events are recorded, for which resources, and with what level of detail.

Each action being performed can generate a series of events, defined as stages:
#### RequestReceived
Generated as soon as the audit handler receives the request
#### ResponseStarted
Generated when the response headers are sent, but before the response body is sent (generated only for long-running requests such as watches)
#### ResponseComplete
Generated when the response body has been completed
#### Panic
Generated when a panic occurs

The level of detail recorded for each event can be one of the following:

#### None
Does not log the event at all
#### Metadata
Logs the request metadata (user, timestamp, resource, verb, etc.) but not the request details or response body
#### Request
Logs event metadata and the request body but not the response body
#### RequestResponse
Logs the full details of the event, including the metadata and request and response bodies

Kubernetes’ audit policy is very flexible and well documented in the main Kubernetes documentation. Included here are just a couple of simple examples to illustrate.

To log all requests at the metadata level:
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```
To omit the RequestReceived stage and to log pod changes at the RequestResponse level and configmap and secrets changes at the metadata level:
```yaml
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]
```
This second example illustrates the important consideration of sensitive data in audit logs. Depending on the level of security around access to the audit logs, it may be essential to ensure the audit policy does not log details of secrets or other sensitive data. Just like the practice of encrypting secrets at rest, it is generally a good practice to always exclude sensitive details from your audit log.
### Restrict Access to Alpha or Beta Features

### Upgrade Kubernetes Frequently



## Use a Managed Kubernetes Service


### CIS Benchmarks
### Network security
This covers securely integrating the cluster with the surrounding infrastructure, and in particular which network interactions between the cluster and the surrounding infrastructure are allowed, for control plane, host, and workload traffic.
