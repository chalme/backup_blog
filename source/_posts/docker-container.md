---
title: 关于容器技术
date: 2016-12-11 01:34:08
tags:
- docker
---

## 关于容器技术

> **容器定义**
> 首先必须是一个相对独立的运行环境，在这一点上，有点类似虚拟机的概念，但又没有虚拟机那样彻底。另外，在一个容器环境内，应该最小化其对外界的影响，比如不能在容器中把host上的资源全部消耗掉，这就是资源控制。

<!-- more -->
- 容器技术已经集成到了Linux内核中，已经被当作Linux内核原生提供的特性。
- 容器技术主要包括`Namespace`和`Cgroup`两个内核特性。
    1. Namespace又称命名空间，它主要做访问隔离。其原理是针对一类资源进行抽象， 并将其封装在一起提供给一个容器使用，对于这类资源，因 为每个容器都有自己的抽象，而它们彼此之间是不可见的，所以就可以 做到访问隔离。
    2. Cgroup 是control group的简称，又称控制组，它主要是做资源控制。 其原理是将一组进程放在一个控制组里，通过给这个控制组分配指定的 可用资源，达到控制这一组进程可用资源的目的。

  实际上，Namespace和Cgroup并不是强相关的两种技术，用户可以根据需要单 独使用他们，比如单独使用Cgroup做资源管理，就是一种比较常见的做法。而 如果把他们应用到一起，在一个Namespace中的进程恰好又在一个Cgroup中， 那么这些进程就既有访问隔离，又有资源控制，符合容器的特性，这样就创建 了一个容器。

## 理解容器

- 容器的核心技术是 Cgroup + Namespace ，但光有这两个抽象的技术概念是无 法组成容器。Linux 容器的最小组成，可以由以下公式来表示：
    ```
    容器 = cgroup + namespace + rootfs + 容器引擎（用户态工具）
    ```
    其各项功能分别是：
    - Cgroup：资源控制。
    - Namespace： 访问控制。
    - rootfs：文件系统隔离。
    - 容器引擎：生命周期控制。

- 容器的创建原理。
    ```
    代码一：
    pid = clone(fun, stack, flags, clone_arg);
    (flags: CLONE_NEWPID | CLONE_NEWNS | CLONE_NEWUSER | CLONE_NEWNET
    | CLONE_NEWipc | CLONE_NEWuts | ...)

    代码二：
    echo $pid > /sys/fs/cgroup/cpu/tasks
    echo $pid > /sys/fs/cgroup/cpuset/tasks
    echo $pid > /sys/fs/cgroup/blkio/tasks
    echo $pid > /sys/fs/cgroup/memory/tasks
    echo $pid > /sys/fs/cgroup/devices/tasks
    echo $pid > /sys/fs/cgroup/freezer/tasks

    代码三：
    fun() {
        ...
        pivot_root("path_of_rootfs/", path);
        ...
        exec("/bin/bash");
        ...
    }
    ```
    1. 代码一，通过clone系统调用，并传入各个Namespace对应的clone flag ，创建了一个新的子进程，该进程拥有自己的Namespace。根据以上代码 可知，该进程拥有自己的pid、mount、user、net、ipc、uts namespace 。
    2. 代码二，将代码一中的进程pid写入各个Cgroup子系统中，这样该进程就 可以受到相应Cgroup子系统的控制。
    3. 代码三，该fun函数由上面生成的新进程执行，在fun函数中，通过 pivot_root系统调用，使进程进入一个新的rootfs，之后通过exec系统 调用，在新的Namespace、Cgroup、 rootfs 中执行"/bin/bash"程序。

    通过以上操作，成功地在一个“容器”中运行了一个bash程序。

## Cgroup 介绍
### Cgroup 是什么
Cgroup 是 control group 的简写，属于Linux内核提供的一个特性，用于限制和隔离一组 进程对系统资源的使用，也就是做资源QoS，这些资源主要包括CPU、内存、block I/O和网 络带宽。
- 从实现的角度来看，Cgroup实现了一个通用的进程分组的框架，而不同资源的具体管理则 是由各个Cgroup子系统实现的。截止到内核4.1版本，Cgroup中实现的子系统及其作用如下：
    1. devices：设备权限控制。
    2. cpuset：分配指定的CPU和内存节点。
    3. cpu：控制CPU占用率。
    4. cpuacct：统计CPU使用情况。
    5. memory：限制内存的使用上限。
    6. freezer：冻结（暂停）Cgroup中的进程。
    7. net_cls：配合tc（traffic controller）限制网络带宽。
    8. net_prio：设置进程的网络流量优先级。
    9. huge_tlb：限制HugeTLB的使用。
    10. perf_event：允许Perf工具基于Cgroup分组做性能检测。
- 在Cgroup出现之前，只能对一个进程做一些资源控制。

## Namespace 介绍
### Namespace 是什么
Namespace 是将内核的全局资源做封装，使得每个Namespace都有一份独立的资源，因此不同的进程在各自的Namespace内对同一种资源的使用不会互相干扰。
- 目前Linux内核总共实现了6种Namespace:
    1. IPC: 隔离System V IPC和POSIX消息队列。
    2. Network：隔离网络资源。
    3. Mount：隔离文件系统挂载点。
    4. PID：隔离进程ID。
    5. UTS：隔离主机名和域名。
    6. User：隔离用户ID和组ID。

## 最后。。
很失望目前不能继续下去。我读的这本书是`docker进阶与实践`，确实是进阶，之前没有了解过docker，现在写的基本都是在摘抄书上的内容，摘抄并没有错，但是确实理解不是特别 深刻，过段时间，在读。
