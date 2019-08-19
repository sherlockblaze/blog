---
title: K8S 编程 -- 开端
tags: 
  - K8S
date: 2019-08-19
---

Controllers can act on core resources such as deployments or services, which are typically part of the K8S controller manager in the control plane, or can watch and manipulate user-defined custom resouces.

Operators are controllers that encode some operational knowledge, such as application lifecycle management, along with the custom resources defined .

The Control Loop

1. Read the state of resources , preferably event-driven
2. Change the state of objects in the cluster or the cluster-external world. For example, launch a pod, create a network endpoint, or query a cloud API.
3. Update status of the resources in step 1 via the API server in etcd.
4. Repeat cycle; return to step 1.



read resource state > change the world > update resource status -> remain the same.



The control loop is depicted in xxx, which shows the typical moving parts, with the main loop of the controller in the middle.

This main loop is continuously running inside of the controller process. This process is usually running inside a pod in the cluster.

From an architectural point of view, a controller typically uses the following data structures:

- Informers
  - Informers watch the desired state of resources in a scalable and sustainable fashion. They also implement a resync mechanism that enforces periodic reconciliation, and is often used to make sure that the cluster state and is often used to make sure that the cluster state cached in memory do not drift.
- Work queues
  - Essentially, a work queue is a component that can be used by the event handler to handle queuing of state changes and help to implement retries. Resources can be requeued in case of errors when updating the world or writing the status, or just because we have to reconsider the resource after some time for other reasons.

**Let's now take a closer look at the control loop, starting with Kubernetes event-driven architecture.**

**Events**

The K8S control plane heavily employs events and the principle of loosely coupled components. 

Kubernetes controllers watch changes to Kubernetes objects in the API server: adds, updates, and removes. When such an event happens, the controller executes its business logic.

For example, in order to launch a pod via a deployment, a number of controllers and other control plane components work together:

1. The deployment controller notices that the user creates a deployment. It creates a replica set in its business logic.
2. The replica set controller notices the new replica set and subsequently runs its business logic, which creates a pod object.
3. The scheduler -- which is also a controller -- notices the pod with an empty `spec.nodeName` field. Its business logic puts the pod in its scheduling queue.
4. Meanwhile the kubelet -- another controller -- notices the new pod. But the new pod's `spec.nodeName` field is empty and therefore does not match the kubelet's node name. It ignores the pod and goes back to sleep.
5. The scheduler takes the pod out of the work queue and schedules it to a node that has enough free resources by updating the `spec.nodeName` field in the pod and writing it to the API server
6. The `kubelet` wakes up again due to the pod update event. It again compares the `spec.nodeName` with its own node name. The names match, and so the `kubelet` starts the containers of the pod and reports back that the containers have been started by writing this information into the pod status, back to the API server.
7. The replica set controller notices the changed pod but has nothing to do.
8. Eventually the pod terminates. The `kubelet` will notice this, get the pod object from the API server and set the "terminated" condition in the pod's status, and write it back to the API server.
9. The replica set controller notices the terminated pod and decides that this pod must be replaced. It deletes the terminated pod on the API server and creates a new one.
10. An so on.

**A number of independent control loops communicate purely through object changes on the API server and events these changes trigger through informers.**

#### Watch events versus the event object

Watch events and the top-level Event object in Kubernetes are two different things:

- Watch events are sent through streaming HTTP connections between the API server and controllers to drive informers.
- The top-level Event object is a resource like pods, deployments, or services, with the special property that it has a time-to-live of an hour and then is purged automatically from `etcd`.

Event objects are merely a user-visible logging mechanism. A number of controllers create these events in order to communicate aspects of their business logic to the user. For example, the `kubelet` reports the lifecycle events for pods.

#### Edge- Versus Level-Driven Triggers

There are two principled options to detect state change:

- Edge-driven triggers
  - At the point in time the state change occurs, a handler is triggered --- for example, from no pod to pod running.
- Level-driven triggers
  - The state is checked at regular intervals and if certain conditions are met, then a handler is triggered.

The latter is a form of polling. It does not scale well with the number of objects, and the latency of controllers noticing changes depends on the interval of polling and how fast the API server can answer. With many asynchronous controllers involved, as described in  "Events", the result is a system that takes a long time to implement the users' desire.

**The former option is much more efficient with many objects.** Then latency mostly depends on the number of worker threads in the controller' s processing events. Hence , Kubernetes is based on events.

In the kubernetes control plane, a number of components change objects on the API server, with each change leading to an event. We call these components **event sources** or **event producers**. On the other hand, in the context of controllers, we're interested in consuming events -- that is, **when and how to react to an event**(via an informer).

![driven](https://sherlockblaze.com/resources/img/profession/k8s/driven.png)

1. An example of an edge-driven-only logic, where potentially the second state change is missed.
2. An example of an edge-triggered logic, which always gets the lastest state when processing an event. In other words, the logic is edge-triggered but level-driven.
3. An example of an edge-triggered, level-driven logic with additional resync.

Strategy 1 does not cope well with missed events, whether because broken networking makes it lose events, or because the controller itself has bugs or some external cloud API was down.

Strategy 2 recovers from those issues when another event is received because it implements its logic based on the latest state in the cluster. In the case of the replica set controller, it will always compare the specified replica count with the running pods in the cluster. When it loses events, it will replace all missing pods the next time a pod update is received.

Strategy 3 adds continuous resync. If no pod events come in, it will at least reconcile every five minutes, even if the application runs very stably and dose not lead to many pod events.

**Kubernetes controllers typically implement the third strategy.**

#### Changing Cluster Objects or the External World

**ReplicaSets** are used in deployments, and the bottom line of the respective controller is: maintain a user-defined number of identical pod replicas.

```go
// manageReplicas checks and updates replicas for the given ReplicaSet
// It does NOT modify filteredPods
// It will requeue the replica set in case of an error while creating/deleting pods
func (rsc *ReplicaSetController) manageReplicas(filteredPods []*v1.Pod, rs *apps.ReplicaSet) error {
    diff := len(filteredPods) - int(*(rs.Spec.Replicas))
    rsKey, err := controller.KeyFunc(rs)
    if err != nil {
        utilruntime.HandleError(
            fmt.Errorf("Couldn't get key fo %v %#v: %v", rsc.Kind, rs, err)
        )
        return err
    }
    if diff < 0 {
        diff *= -1
        if diff > rsc.burstReplicas {
            diff = rsc.burstReplicas
        }
        rsc.expectations.ExpectCreations(rsKey, diff)
        klog.V(2).Infof("Too few replicas for %v %s/%s, need %d, creating %d", rsc.Kind, rs.Namespace, rs.Name, *(rs.Spec.Replicas), diff)
        successfulCreations, err := slowStartBatch(
            diff,
            controller,SlowStartInitialBatchSize,
            func() error {
                ref := metav1.NewControollerRef(rs, rsc.GroupVersionKind)
                err := rsc.podControl.CreatePodsWithContollerRef(
                    rs.Namespace, &rs.Spec.Template, rs, ref
                )
                if err != nil {
                    return nil
                }
                return err
            },
        )
        if skippedPods := diff - successfulCreations; skippedPods > 0 {
            klog.V(2).Infof("Slow-start failure. Skipping creation of %d pods," + " decrementing expectations for %v %v/%v",
            skippedPods, rsc.Kind, rs.Namespace, rs.Name)
            for i := 0; i < skippedPods; i++ {
                rsc.expectations.CreationObserved(rsKey)
            }
        }
        return err
    } else if diff > 0 {
        if diff > rsc.burstReplicas {
            diff = rsc.burstReplicas
        }
        klog.V(2).Infof("Too many replicas for %v %s/%s, need %d, deleting %d", rsc.Kind, rs.Namespace, rs.Name, *(rs.Spec.Replicas), diff)
        
        podsToDelete := getPodsToDelete(filteredPods, diff)
        rsc.expectations.ExpectDeletions(rsKey, getPodKeys(podsToDelete))
        errCh := make(chan error, diff)
        var wg sync.WaitGroup
        wg.Add(diff)
        for _, pod := range podsToDelete {
            go func(targetPod *v1.Pod) {
                defer wg.Done()
                if err := rsc.podControl.DeletePod(
                    rs.Namespace,
                    targetPod.Name,
                    rs,
                ); err != nil {
                    podKey := controller.PodKey(targetPod)
                    klog.V(2).Infof("Failed to delete %v, decrementing " + "expectations for %v %s/%s", podKey, rsc.Kind, rs.Namespace, rs.Name)
                    rsc.expectations.DeletionObserved(rsKey, podKey)
                    errCh <- err
                }
            }(pod)
        }
        wg.Wait()
        
        select {
        case err := <-errCh:
            if err != nil {
                return err
            }
        default:
        }
    }
    return nil
}
```

- diff < 0: Too few replicas; more pods must be created
- diff > 0: Too many replicas; pods must be deleted.

#### Optimistic Concurrency

![scheduling-architectures-in-distributed-systems](https://sherlockblaze.com/resources/img/profession/k8s/scheduling-architectures-in-distributed-systems.png)

> Our solution is a new parallel scheduler architecture built around shared state, using lock-free optimistic concurrency control, to achieve both implementation extensibility and performance scalability.

While Kubernetes inherited a lot of traits and lessons learned from Borg, this specific, transactional control plane feature comes from Omega: **in order to carry out concurrent operations without locks, the Kubernetes API server uses optimistic concurrency.**

In a nutshell, **that if and when the API server detects concurrent write attempts, it rejects the latter of the two write operations.** It is then up to the client to handle a conflict and potentially retry the write operation.

```go
var err error
for retries := 0; retries < 10; retries++ {
    foo, err = client.Get("foo", metavl.GetOptions{})
    if err != nil {
        break
    }
    <update-the-world-and-foo>
    _, err = client.Update(foo)
    if err != nil && errors.IsConflict(err) {
        continue
    } else if err != nil {
        break
    }
}
```

The code shows a retry loop that gets the latest object `foo` in each iteration, then tries to update the world and `foo's` status to match `foo's` spec. The changes done before the `Update` call are optimistic.

The returned object `foo` from the `client.Get` call contains a resource version, which will tell `etcd` on the write operation behind the `client.Update` call that another actor in the cluster wrote the `foo` object in the meantime. If that's the case, our retry loop will get a resource version conflict error. This means that the optimistic concurrency logic failed. In other words, the `client.Update` call is also optimistic.

```yaml
kind: Pod
metadata:
    name: foo
    resourceVersion: 57
spec:
    ...
status:
    ...
```

Assume the controller needs several seconds with its updates to the world. Seven seconds later, it tries to update the pod it read -- for example, it sets an annotation. Meanwhile, the `kubelet` has noticed yet another container restart and updated the pod's status to reflect that; that is, `resourceVersion` has increased to 58.

The object your controller sends in the update request has `resourceVersion: 57`.

*The essence of this is: conflict errors are totally normal in controllers. Always expect them and handle them gracefully.*

It's important to point out that optimistic concurrency is a perfect fit for level-based logic, **because by using level-based logic you can just rerun the control loop.** Another run of that loop will automatically undo optimistic change from the previous failed optimistic attempt, and it will try to update the world to the latest state.

#### Operators

