---
title: K8S 编程 -- 开端
tags: 
  - K8S
date: 2019-08-19
---

## Controller versus Operator

控制器可以作用于一些核心资源，或者用户自定义的资源。核心资源的话比如说 K8S 里的 deployments 和 services。

操控者是具备了一些操作知识的控制器，比如说一些应用生命周期的管理。

## Controller

### 控制循环

1. 读取资源状态，一般是事件驱动的形式
2. 修改集群中对象的状态，或者集群外的世界。比如，运行一个 pod， 创建一个网络端点，或者查询一个云端 API
3. 更新 etcd 中存放的资源状态
4. 重复循环，回到第一步

> 读取 -> 修改 -> 更新 -> 重复

![k8s control loop](https://sherlockblaze.com/resources/img/profession/k8s/k8s-control-loop.png)

如上图所示，展现一个常规的过程，控制循环在中间位置，且控制循环持续运行在控制进程里，这个过程通常运行在集群里的 pod 中。

### 控制器中的数据结构

从架构的角度来看，控制器通常使用以下的数据结构:

- Informers: Informers 通常以可扩展和可持续的方式观察所需的资源状态。同时也实现了**重新同步机制**，用于实施定期对账，确保在缓存中的集群状态不会有偏差
- Work Queues(工作队列): 工作队列本质上是事件处理器用来处理状态更新排队并实现重试的组件，在状态更新发生错误的时候，可以重新入队等待下次更新

### Events

K8S 控制面板大量使用事件和松耦合组件。

K8S 控制器监视 API 服务器中 K8S 对象的修改: 新增，更新以及移除。当有这样的事件发生时，控制器就会开始干它的活。

1. Deployment 控制器注意到用户创建了一个 Deployment，然后它跟着创建一个 Replica Set
2. Replica Set 控制器注意到 Replica Set 的创建，然后开始创建定义好的 Pod
3. 调度器(其实也是控制器)注意到新创建的 Pod 中 `spec.nodeName` 字段是空的。它的本职工作就是把它放到计划队列中
4. 与此同时，另一个控制器 `kubelet` 注意到这个新的 Pod(通过通知的方式)，但是它注意到这个 Pod 的 `spec.nodeName` 是空的，和想象中不一样，于是就是休眠了，一直到下一个事件发生
5. 调度器把这个 Pod 从工作队列里面拿出来，然后找一个能容得下它的节点，更新它的 `spec.nodeName` 字段为该节点的名字，而且把这件事告诉 API Server
6. 因为这个 Pod 的更新(事件)，`kubelet` 被惊醒，然后它开始比较 `spec.nodeName` 字段和自己管辖节点的名称。如果名字匹配了，那它就启动 pod 中定义的容器，然后把这个事情报告回去(通过更新 pod 的状态，并返回给 API Server 的方式)
7. Replica Set 控制器注意到这个 Pod 的改变，但无动于衷
8. 最后 Pod 运行结束了，`kubelet` 会注意到这个，从 API Server 手上拿到这个 Pod 对象，并且设置它的状态为 *terminated*，然后写回 API Server
9. Replica Set 控制器注意到这个死掉的 Pod，并且决定要替换这个 Pod。它通过 API Server 删掉这个死掉的 Pod，然后创建一个新的
10. 重复

**可以发现，有很多独立的控制循环，单纯的通过 API 服务器上的对向更改和这些更改通过触发器触发的事件进行通信**

### 监听事件和事件对象

K8S 里面，监听事件和事件对象是两个不同的事:

- 监听事件通过 API Server 和控制器之间的流式 HTTP 连接发送，来驱动线程
- 事件对象是诸如 Pods，Deployments 或者 Services 这样的资源，具有特殊属性，这些属性具有一小时的生存时间，然后从 `etcd` 自动清除

事件对象仅仅是用户课件的日志记录机制，许多控制器创建这些事件，以便将其业务逻辑的各个方面传达给用户。例如，`kubelet` 报告 pod 的生命周期事件

### 边缘驱动和水平驱动触发器

有两个检测状态变化的方式：

- 边缘驱动触发器: 在事情发生的那个点，触发处理程序，例如，从没有 pod 到有 pod 运行的那个时刻
- 水平驱动触发器: 定期检查状态，如果满足某些条件(例如，pod 运行)，就触发处理程序

后者是轮训的方式，它不能很好的适应对象的数量，并且控制器注意到更改的延迟取决于轮询的间隔以及 API Server 应答的速度。如"事件"中描述的一样，因为涉及到许多异步控制器，导致需要很长时间来满足用户的期望。

**前者的方式在处理很多对象的时相对更加有效。**这种情况下延迟主要取巨额月控制器处理事件中的工作线程数。所以，K8S选择基于事件(边缘驱动的触发器)

K8S 控制面板中，有很多组件通过 API Server 修改对象，每次修改都会触发一次事件。我们称呼这个组件为 **病原体(事件源)**或者**搞事者(事件生产者)**。问题是，我们怎么去响应这些事件(通过**informer(告密者)**)。

![driven](https://sherlockblaze.com/resources/img/profession/k8s/driven.png)

1. 第一个例子是一个简单的事件驱动逻辑，有第二种状态更新丢失的潜在可能
2. 第二个例子是事件触发的逻辑，搭配了轮训的方式，这样总是可以获得最终的状态，本质上来说，逻辑上还是轮训
3. 最后一个例子，结合了事件触发和轮训的方式，同时配合了重新同步的机制


第一种策略对丢失事件的处理不是很好，如果网络出现故障或者因为控制器自身的 Bug 或者什么原因导致了事件的丢失，会影响集群的状态

第二种策略下，当有事件丢失时，控制器会通过轮训的方式去集群中比对状态，并进行更新。举个例子 Replica Set 控制器，它始终将指定的副本计数和集群中正在运行的 pod 进行比较，当它丢失事件时，会在下次收到 pod 更新时替换所有丢失的 pod

第三种策略添加了持续的重新同步。即使应用程序运行的非常稳定且不会导致许多 pod 事件，为了稳，它也会至少每五分钟协调一次。

**K8S 控制器一般来说使用第三种策略**

### 更新集群内对象

**ReplicaSet** 控制器用于 **Deployments** 控制器，并且相应控制器的基本职责是: 维护用户定义数量的相同 Pod 副本

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

- diff < 0: 副本数少了，更多的 Pod 需要被创建
- diff > 0: 副本数多了，有些 Pod 需要被删除

### 乐观并发

![scheduling-architectures-in-distributed-systems](https://sherlockblaze.com/resources/img/profession/k8s/scheduling-architectures-in-distributed-systems.png)

> 我们的解决方案是围绕共享状态勾线新的并行调度程序体系结构，使用乐观并发(无锁)来控制实现可扩展性和性能伸缩性

为了在无锁的情况下执行并发操作，K8S API Server 使用了乐观并发

简单来说，**当有API Server 检测到两个同时的写请求，会拒绝两个里面稍微发出请求的那个。**剩下的事情就交由客户端自己处理了，重新发起写操作

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

以上代码展示了一个重试逻辑，在每个迭代中去获取对象 `foo` 的最终状态，然后尝试去更新对象 `foo` 的状态。在 Update 调用之前完成的更改是乐观状态的

从 `client.Get` 中返回的 `foo` 对象是源版本，当要使用 `client.Update` 去更新时，如果此时集群中有另外一个搞事的也要去修改同一个对象(`foo`)，重试逻辑或得到一个报错(冲突)，即操作失败了。用另一句话说，就是 `client.Update` 调用也是乐观的

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

假设控制器需要几秒钟去修改一个对象，7s 钟后，它尝试去修改读取到的 pod(如上描述)。在同一个事件，`kubelet` 控制器注意到另一个容器重新启动并更新 pod 的状态，这个时候 `resourceVersion` 变成了 58

但是你的控制器发送请求时携带的 `resourceVersion` 参数值仍然是 57

*实质是: 控制器中冲突出现是很正常的，需要期待它们可以很 nice 的处理好这些问题*

非常重要的一点是，需要知道乐观并发非常适合用在轮询逻辑里，**因为通过基于轮询的逻辑，你可以重新运行控制循环，然后再来一次之前的过程。**

## Operators

**Operator 是一个特定于应用程序的控制器，它扩展了 K8S API，以代表 K8S 用户创建，配置和管理复杂有状态应用程序的实例。**

**An Operator is an application-specific controller that extends the Kubernetes API to create, configure, and manage instances of complex stateful applications on behalf of a Kubernetes user.**  It builds upon the basic Kubernetes resources and controller concepts but includes domain or application-specific knowledge to automate common tasks.

- There's some domain-specific operational knowledge you'd like to automate
- The best practices for this operational knowledge are known and can be made explicit -- for example, in the case of a Cassandra operator, when and how to re-balance nodes, or in the case of an operator for a service mesh, how to create a route.
- The artifacts shipped in the context of the operator are:
  - A set of custom resource definitions(CRDs) capturing the domain-specific schema and custom resources following the CRDs that, on the instance level, represent the domain of interest.
  - A custom controller, supervising the custom resources, potentially along with core resources. For example, the custom controller might spin up a pod.

![concept-of-an-operator](https://sherlockblaze.com/resources/img/profession/k8s/concept-of-an-operator.png)