---
title: Rebalancing cluster workloads
shortTitle: Rebalance workloads
intro: 'You can force your {% data variables.product.prodname_ghe_server %} cluster to evenly distribute job allocations for workloads on the cluster''s nodes.'
product: '{% data reusables.gated-features.cluster %}'
permissions: 'People with administrative SSH access to a {% data variables.product.prodname_ghe_server %} instance can rebalance cluster workloads on the instance.'
versions:
  ghes: '*'
type: how_to
topics:
  - Clustering
  - Enterprise
redirect_from:
  - /admin/enterprise-management/configuring-clustering/rebalancing-cluster-workloads
  - /admin/monitoring-managing-and-updating-your-instance/configuring-clustering/rebalancing-cluster-workloads
---

## About workload balance for a {% data variables.product.prodname_ghe_server %} cluster

A {% data variables.product.prodname_ghe_server %} instance in a cluster configuration assigns each task to a node according to the node's role. This assignment is called an allocation.

If a cluster node is unreachable by other nodes due to a hardware or software failure, your instance creates a new allocation to distribute jobs from the unhealthy node to another node that can handle the workload. In some situations, this distribution does not occur automatically, and a single node may run more jobs than expected.

You can manage allocations using the `ghe-cluster-balance` utility, which can display the status of existing allocations or force your instance to balance allocations. For example, you should balance allocations after you add a new node to the cluster. Optionally, you can schedule regular balancing.

You can run the following commands from any node in your cluster using the administrative shell. For more information, see [Accessing the administrative shell (SSH)](/admin/configuration/configuring-your-enterprise/accessing-the-administrative-shell-ssh).

## Checking the distribution of cluster jobs

In some cases, such as hardware failure, the underlying software that manages allocations will migrate tasks from the unhealthy node to a healthy node. If the unhealthy node recovers, the task may remain assigned to the recovered node, which can result in unbalanced load. The risk of job failure may increase if allocations are unbalanced and additional nodes fail. You can check the distribution of allocations using the `ghe-cluster-balance status` utility.

1. To see a list of allocations, run the following command. The utility displays healthy allocations in green. If any jobs are not properly distributed, the utility displays the allocation's count in red.

   ```shell copy
   ghe-cluster-balance status
   ```

1. If a job is not properly distributed, inspect the allocations by running the following command. Replace JOB with a single job or comma-delimited list of jobs.

   ```shell copy
    ghe-cluster-balance status -j JOB
   ```

   For example, to see the status of allocations for your instance's HTTP server and authorization service, you can run `ghe-cluster-balance status -j github-unicorn,authzd`.

## Rebalancing allocations

After you determine which jobs are unbalanced across your cluster's nodes, you can rebalance allocations using the `ghe-cluster-balance rebalance` utility. The utility checks the distribution of existing jobs. If any jobs are unbalanced, the utility displays the jobs and prompts you to continue. If you continue, the utility creates new allocations to redistribute the jobs.

1. To perform a dry run and see the result of rebalancing without making changes, run the following command. Replace JOB with a single job or comma-delimited list of jobs.

   ```shell copy
   ghe-cluster-balance rebalance --dry-run -j JOB
   ```

   For example, to perform a dry run of rebalancing jobs for your instance's HTTP server and authorization service, you can run `ghe-cluster-balance rebalance --dry-run -j github-unicorn,authzd`.
1. To rebalance, run the following command. Replace JOB with a single job or comma-delimited list of jobs.

   ```shell copy
   ghe-cluster-balance rebalance -j JOB
   ```

## Scheduling allocation rebalancing

You can schedule rebalancing of jobs on your cluster by setting and applying configuration values for {% data variables.location.product_location %}.

> [!NOTE]
> Currently, you can only schedule reallocation of jobs for the HTTP server, `github-unicorn`.

1. To configure automatic, hourly balancing of jobs, run the following command.

   ```shell copy
   ghe-config app.cluster-rebalance.enabled true
   ```

1. Optionally, you can override the default schedule by defining a [Systemd.time expression](https://www.freedesktop.org/software/systemd/man/latest/systemd.time.html). For example, run the following command to balance jobs daily.

   ```shell copy
   ghe-config app.cluster-rebalance.schedule 'daily'
   ```

{% data reusables.enterprise.apply-configuration %}

## Further reading

* [AUTOTITLE](/admin/configuration/configuring-your-enterprise/command-line-utilities#ghe-cluster-balance)
