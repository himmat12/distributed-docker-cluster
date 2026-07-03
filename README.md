# Distributed Docker Cluster
A distributed docker containers cluster for experimenting with distributed computing and parallel workloads using Docker Compose, Celery, and Redis.

## Problem statement
Given a Celery worker executing a computationally heavy task, where picking an integer $k$ from an array requires generating a sequence of the first $k$ perfect squares, with a simulated $2\text{-second}$ latency imposed on *each* individual square calculation, the workload per task scales dynamically based on the value of $k$ ($k \times 2\text{ seconds}$).

If the requirement is to process an array of size $n$ containing various integers $[k_1, k_2, ..., k_n]$, a sequential execution results in a total latency of $\left(\sum k\right) \times 2\text{ seconds}$. To optimize performance and prevent long-running tasks from blocking the pipeline, how can we distribute these dynamic workloads across a cluster of multiple, horizontally scaled Docker container nodes?

## System Architecture (Producer/Consumer)

```txt
+-----------------------------------------------------------------+
|                       LAPTOP / HOST MACHINE                     |
|                                                                 |
|   +------------------+                                          |
|   | PRODUCER         | (Container: producer)                    |
|   | [Array of size n]|                                          |
|   +--------+---------+                                          |
|            |                                                    |
|            | 1. Dispatches 'n' asynchronous tasks               |
|            v                                                    |
|   +------------------+                                          |
|   | MESSAGE BROKER   | (Container: redis)                       |
|   | [Task Queue]     |                                          |
|   +--------+---------+                                          |
|            |                                                    |
|            +-------------------+-------------------+            |
|            | 2. Pulls tasks    |                   |            |
|            v                   v                   v            |
|   +--------------+   +--------------+   +--------------+        |
|   | COMPUTATION  |   | COMPUTATION  |   | COMPUTATION  |        |
|   | NODE 1       |   | NODE 2       |   | NODE 'M'     |        |
|   | (worker_1)   |   | (worker_2)   |   | (worker_m)   |        |
|   +--------------+   +--------------+   +--------------+        |
|                                                                 |
+-----------------------------------------------------------------+
```