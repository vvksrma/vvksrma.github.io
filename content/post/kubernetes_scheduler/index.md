---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Kubernetes Scheduler module"
subtitle: ""
summary: "How the scheduler module of kubernetes works, a code review"
authors: [Pratham Sahu]
tags: []
categories: []
date: 2024-02-23T13:34:30+05:30
lastmod: 2024-02-23T13:34:30+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
# Kubernetes Scheduling Module

## Kubernetes Architecture

Kubernetes is an orchestration platform for efficient management of microservices in a distributed system setup. I have personally used Kubernetes to deploy the cryptographically secure dating platform at IIT Kanpur.(PuppyLove)


You can find my blog about it [here](https://prathamsahu52.github.io/post/puppylove2.0_deployment/)

Let us first talk about the high level architecture of a Kubernetes Cluster and how it interacts with th external world using kubectl API.

![image](https://hackmd.io/_uploads/B1oDqj3j6.png)
Reference: https://kubernetes.io/docs/concepts/architecture/

There are multiple nodes within a given cluster which run `pods` within it. Pods are containers that can be spawned using docker images.
Kubernetes follows a client-server architecture model where the `CONTROL PLANE` acts as the master node and controls the working of the servant nodes which run the processes. You can assume it's functionality analogous to an OS in a system where the entire state of the cluster is managed by the `CONTROL PLANE`.

That should be more than enough context for the next part, where we deep dive into the Kubernetes Scheduler. You can read more about the architecture [here](https://kubernetes.io/docs/concepts/architecture/).


## Scheduler

As per the kubernetes documentation, 

<mark>In Kubernetes, scheduling refers to making sure that Pods are matched to Nodes so that the kubelet can run them. Preemption is the process of terminating Pods with lower Priority so that Pods with higher Priority can schedule on Nodes. Eviction is the process of terminating one or more Pods on Nodes.</mark>


We will take a look into the following directory to understand the working in a more better way.

[/pkg/scheduler](https://github.com/kubernetes/kubernetes/tree/91ee30074bee617d52fc24dc85132fe948aa5153/pkg/scheduler)


The highlevel view of the scheduler is presented here

![image](https://hackmd.io/_uploads/rJK_X32oa.png)
Reference: https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/


The scheduling cycle selects a node for the Pod, and the binding cycle applies that decision to the cluster. The scheduler instance has a loop running indefinitely which (everytime there’s a pod) is responsible for invoking the scheduling logic and making sure a pod gets either a node assigned or requeued for future processing. Each loop consists of a blocking scheduling and a non-blocking binding cycle.

Let us walk through how the scheduler instance works

### Scheduler instance is initialised

The scheduler is initialised [here](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L190). It does the following tasks:

- [scheduler cache]() is initialised.
- Initialises the scheduler with options
- Configurator builds the instance( connects scheduler algorithm, caches, plugins)
- Event handlers are registered which allow the scheduler to handle changes from external api calls.


### Scheduling a pod

[scheduleOne](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L441) This process is serialised. It is used to 
It gets the next pod to be scheduled using `podInfo := sched.NextPod()`. It then calls the Scheduler Algorithm using `scheduleResult, err := sched.Algorithm.Schedule(schedulingCycleCtx, fwk, state, pod)`

Once the pod is scheduled, it adds it to the list of `AssumePods`. These are pods which have been scheduled to a node but are not bound to the node yet. This calls `assume` function whose code can be found [here](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L373).

This function adds this AssumePod to the cache. The cache is used to free the scheduling context for the next pod until the binding cycle returns with `FinishBinding` signal.
The working of the Scheduler Cache is explained in detail in the next session. 
The scheduling cycle then calls the Permit Plugin.

- Permit Plugin: Permit plugins are invoked at the end of the scheduling cycle for each Pod, to prevent or delay the binding to the candidate node. It can either return approve, deny or wait.



### Scheduler Cache

[code](https://github.com/kubernetes/kubernetes/blob/master/pkg/scheduler/internal/cache/cache.go)
Cache is responsible for capturing the current state of a cluster.Allowing to update the snapshot of a cluster (to pin the cluster state while a scheduling algorithm is run) with the latest state at the beginning of each scheduling cycle.

The cache also allows to run [assume](https://github.com/kubernetes/kubernetes/blob/91ee30074bee617d52fc24dc85132fe948aa5153/pkg/scheduler/internal/cache/cache.go#L360) operation which temporarily stores a pod in the cache and makes it look as the pod is actually already running on a designated node for all consumers of the snapshot. It thus increases the throughput of the Scheduler.




### Binding the pod to the node

It binds the pods to the node and returns a signal to the Schedular Cache when done. It consists of 4 stages:
Consists of the following four steps ran in the same order:
- Invoking [WaitOnPermit](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L560)(internal API) of plugins from `Permit` extension point. Some plugins from the extension point may send a request for an operation requiring to wait for a condition (e.g. wait for additional resources to be available or wait for all pods in a gang to be assumed).Under the hood, `WaitOnPermit` waits for such a condition to be met within a timeout threshold.
- Invoking plugins from [PreBind](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L580) extension point
- Invoking plugins from [Bind](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L592) extension point
- Invoking plugins from [PostBind](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/scheduler.go#L611) extension point 





### Scheduling Framework

[code](https://github.com/kubernetes/kubernetes/tree/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/framework)

It is used to filter and score the nodes on which the pods are to be scheduled. The plugins are intitialized [here](https://github.com/kubernetes/kubernetes/blob/4740173f3378ef9d0dc59b0aa9299444a97d0818/pkg/scheduler/framework/runtime/framework.go#L310) by passing a [framework handler](https://github.com/kubernetes/kubernetes/blob/4740173f3378ef9d0dc59b0aa9299444a97d0818/pkg/scheduler/framework/runtime/framework.go#L245), which provides interfaces to manage the pods, nodes and query other handlers.


The scheduler interfaces the [Scheduling Algorithm](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/core/generic_scheduler.go#L61-L66). The algorithm does the following:

- [Take snapshot from cache](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/core/generic_scheduler.go#L101)
- Find out the nodes on which the pod can be scheduled. [code](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/core/generic_scheduler.go#L110).
- If there are atleast two nodes which the pod can be scheduled on, using a [scoring mechanism](https://github.com/kubernetes/kubernetes/blob/a651804427dd9a15bb91e1c4fb7a79994e4817a2/pkg/scheduler/core/generic_scheduler.go#L133) to prioritize.
### Queueing Mechanism

The queueing mechanism allows the scheduler to pick the best pod for the next scheduling cycle. A pod can have various dependencies to fulfill(ex: Persistant Volumes, Config Maps, etc), so the scheduler needs to be able to postpone its scheduling until it can be successfully scheduled.

The scheduler maintains 3 queues:
- active ([activeQ](https://github.com/kubernetes/kubernetes/blob/4cc1127e9251fff364d5c77e2a9a9c3ad42383ab/pkg/scheduler/internal/queue/scheduling_queue.go#L130)): providing pods for immediate scheduling. It is implemented as a heap and is the highest priority qeueue.
- unschedulable ([unschedulableQ](https://github.com/kubernetes/kubernetes/blob/4cc1127e9251fff364d5c77e2a9a9c3ad42383ab/pkg/scheduler/internal/queue/scheduling_queue.go#L135)): for parking pods which are waiting for certain condition(s) to happen. It is implemented as a map.
- backoff ([podBackoffQ](https://github.com/kubernetes/kubernetes/blob/4cc1127e9251fff364d5c77e2a9a9c3ad42383ab/pkg/scheduler/internal/queue/scheduling_queue.go#L133)): exponentially postponing pods which failed to be scheduled (e.g. volume still getting created) but are expected to get scheduled eventually. It is implemented as a queue. It also sets a timeout to try again, whose value increases exponentially until a threshhold.


### Suggested Improvement Ideas:

- Advanced Resource Matching:

Implement more granular resource matching and scheduling based on detailed resource requirements (CPU, memory, I/O bandwidth, etc.) and node characteristics. 

- Predictive Scheduling:

Incorporate machine learning or statistical models to predict future resource requirements and usage patterns, allowing the scheduler to preemptively make decisions that optimize resource utilization and application performance.

