Troubleshooting
===============

This document contains frequently asked troubleshooting problems.

Unable to connect to scheduler
------------------------------

The most common issue is not being able to connect to the cluster once it has been constructed.

Each cluster manager will construct a Dask scheduler and by default expose it via a public IP address. You must be able
to connect to that address on ports ``8786`` and ``8787`` from wherver your Python session is.

If you are unable to connect to this address it is likely that there is something wrong with your network configuration,
for example you may have corporate policies implementing additional firewall rules on your account.

To reduce the chances of this happening it is often simplest to run Dask Cloudprovider from within the cloud you are trying
to use and configure private networking only. See your specific cluster manager docs for info.

Invalid CPU or Memory
---------------------

When working with ``FargateCluster`` or ``ECSCluster``, CPU and memory arguments can only take values from a fixed set of combinations.

So, for example, code like this will result in an error

.. code-block:: python

    from dask_cloudprovider import FargateCluster
    cluster = FargateCluster(
        image="daskdev/dask:latest",
        worker_cpu=256,
        worker_mem=30720,
        n_workers=2,
        fargate_use_private_ip=False,
        scheduler_timeout="15 minutes"
    )
    client = Client(cluster)
    cluster

    # botocore.errorfactory.ClientException:
    # An error occurred (ClientException) when calling the RegisterTaskDefinition operation:
    # No Fargate configuration exists for given values.


This is because ECS and Fargate task definitions with ``CPU=256`` cannot have as much memory as that code is requesting.

The AWS-accepted set of combinations is documented at
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html.

Requested CPU Configuration Above Limit
---------------------------------------
When creating a ``FargateCluster`` or or ``ECSCluster``, or adding additional workers, you may receive an error response with
"The requested CPU configuration is above your limit". This means that the scheduler and workers requested and any other
EC2 resources you have running in that region use up more than your current service quota
`limit for vCPUs <https://aws.amazon.com/ec2/faqs/#EC2_On-Demand_Instance_limits>`_.

You can adjust the scheduler and/or worker CPUs with the ``scheduler_cpu`` and ``worker_cpu``
`arguments <https://cloudprovider.dask.org/en/latest/aws.html#elastic-container-service-ecs>`_. See the "Invalid CPU or Memory"
section in this document for more information.

However, to get the desired cluster configuration you'll need to request a service limit quota increase.

Go to ``https://<region>.aws.amazon.com/servicequotas/home/services/ec2/quotas`` and
`request an increase <https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html>`_ for
"Running On-Demand Standard (A, C, D, H, I, M, R, T, Z) instances".