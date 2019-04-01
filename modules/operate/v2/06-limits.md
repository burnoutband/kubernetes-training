# Pod Limits

## Module Objectives

1. Set pod limits

---

## Set Pod Limits

Currently, all our Pods can consume as much resources as they want. This is rarely a good idea as our Pods can influence somebody else's workloads (noisy-neighbour).

First, let's verify that right now this is the case.

1. Get inside the `backend` pod

    ```shell
    kubectl exec -i -t backend bash
    ```

1. Check how much memory is available.

    ```shell
    cat /proc/meminfo
    ```

    ```
    MemTotal:        3794356 kB
    MemFree:          864724 kB
    MemAvailable:    2665768 kB
    ...
    ```

1. Install `stress` utility inside the container.

    ```shell
    apt-get update && apt-get install stress
    ```

1. Try to consume 1GB of memory.

    ```shell
    stress --vm-bytes 1g --vm-keep -m 1
    ```

1. In a different terminal window, exec into the `backend` pod again and run the `top` command.

    ```shell
    Tasks:   7 total,   2 running,   5 sleeping,   0 stopped,   0 zombie
    %Cpu(s): 99.0 us,  1.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem :  3794356 total,   114472 free,  1937036 used,  1742848 buff/cache
    KiB Swap:        0 total,        0 free,        0 used.  1612368 avail Mem

      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
      237 root      20   0 1055860 1.000g    212 R 96.0 27.6   1:54.22 stress
        1 root      20   0    4292    732    656 S  0.0  0.0   0:00.00 sh
        5 root      20   0   52020   8520   6500 S  0.0  0.2   0:00.04 app
       10 root      20   0   18208   3200   2644 S  0.0  0.1   0:00.02 bash
      236 root      20   0    7280    896    816 S  0.0  0.0   0:00.00 stress
      238 root      20   0   18204   3392   2832 S  0.0  0.1   0:00.00 bash
      244 root      20   0   41032   3124   2660 R  0.0  0.1   0:00.01 top
    ```

1. Stop the `stress` and `top` commands and exit from the `backend` Pod in both terminal windows.

    Now let's try to prevent the Pod from consuming as much memory as it wants.

1. Add the following section to the `manifests/backend.yaml` inline with the other container properties.

    ```yaml
        resources:
          limits:
            memory: "800Mi"
          requests:
            memory: "600Mi"
    ```

    We define resource limits for all containers individually, so this element should go under `spec -> containers[name=backend]` and should be aligned together with `image` and `command` properties.

1. Delete and recreate the `backend` Pod.

1. Exec into the `backend` Pod and install stress again.

1. Try to run the same `stress` command inside the `backend` Pod again.

    ```shell
    stress --vm-bytes 1g --vm-keep -m 1
    ```

    ```
    stress: info: [226] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
    stress: FAIL: [226] (415) <-- worker 227 got signal 9
    stress: WARN: [226] (417) now reaping child worker processes
    stress: FAIL: [226] (451) failed run completed in 1s
    ```
