# Blacksmith RabbitMQ Forge

This Blacksmith Forge teaches a [Blacksmith Broker][broker] how to
deploy standalone and clustered [RabbitMQ][rabbitmq] service
deployments for message bus applications.

## Deploying

To deploy this forge, you will need to add it to your existing
Blacksmith Broker manifest deployment, co-locating the
`rabbitmq-blacksmith-plans` job on the Blacksmith instance group.

Here's an example to get you started (clipped for brevity):

```yaml
releases:
  - name:    rabbitmq-forge
    version: latest

instance_groups:
  - name: blacksmith
    jobs:
      - name:    rabbitmq-blacksmith-plans
        release: rabbitmq-forge
        properties:
          plans:
            # your plans here
            # (see below)
```

The RabbitMQ Forge deploys rabbitmq by using jobs that are found
_inside_ the `rabbitmq-forge` BOSH release, which means that your
Blacksmith BOSH director also needs that release.  Blacksmith is
able to upload that release for you, if you want.

For the Spruce users out there:

```
---
instance_groups:
  - name: blacksmith
    jobs:
      - name: blacksmith
        properties:
          releases:
            - (( append ))
            - (( grab releases.rabbitmq-forge ))
```

Finally, you'll need to define plans for Blacksmith to deploy.
The following sections discuss those ad nauseum.

## Standalone Topology

The `standalone` topology is as straightforward as they come: a
singe dedicated VM that runs a single RabbitMQ process, bound on
all interfaces, to port 5672, with a management UI on 15672.

Here's a diagram to clear things up:

![Standalone Topology Diagram](docs/diag/topology-standalone.png)

### Configuration Options

- *vm_type* - The name of a BOSH `vm_type` from your cloud-config.
  You can use this to size your RabbitMQ appropriate to your workload
  requirements, in terms of RAM and CPU.  Increasing the disk size
  via the VM type is not going to net you much of a gain (see
  the `disk_size` and `persist` options instead.

- *network* - The name of the network to deply these instances to.
  This network should be defined in your cloud-config, and should
  be large enough to handle your anticipated service footprint.
  It does not need any static IP addresses.

  By default, VMs will be deployed into a network named
  `rabbitmq-service`.

### Example Configuration

A single standalone plan:

```
instance_groups:
  - name: blacksmith
    jobs:
      - name:    rabbitmq-blacksmith-plans
        release: rabbitmq-forge
        properties:
          plans:
            dedicated:
              type: standalone
```

## Clustered Topology

The `cluster` topology scales up the RabbitMQ instances and sets
them up in a message bus cluster, striping across BOSH
availability zones.

Here's a diagram:

![Cluster Topology Diagram](docs/diag/topology-cluster.png)

### Configuration Options

- *vm_type* - The name of a BOSH `vm_type` from your cloud-config.
  You can use this to size your RabbitMQ appropriate to your workload
  requirements, in terms of RAM and CPU.

- *azs* - A list of BOSH availability zone names (per
  cloud-config), across which to stripe the nodes.  By default,
  nodes will be put in `z1` and `z2`.

- *network* - The name of the network to deply these instances to.
  This network should be defined in your cloud-config, and should
  be large enough to handle your anticipated service footprint.
  It does not need any static IP addresses.

  By default, VMs will be deployed into a network named
  `rabbitmq-service`.

## Example Configuration

Here's the configuration for the cluster pictured in the topology
digram above:

```
instance_groups:
  - name: blacksmith
    jobs:
      - name:    rabbitmq-blacksmith-plans
        release: rabbitmq-forge
        properties:
          plans:
            clustered:
              type:      cluster
              instances: 3
```

Here, we provide two different clustered configurations, one with
lots of nodes, and another with very large (per cloud-config)
nodes.

```
instance_groups:
  - name: blacksmith
    jobs:
      - name:    rabbitmq-blacksmith-plans
        release: rabbitmq-forge
        properties:
          plans:
            wide:
              type:      cluster
              instances: 7

            large:
              type:     cluster
              vm_type:  very-large
```

## Contributing

If you find a bug, please raise a [Github Issue][1] first,
before submitting a PR.




[1]: https://github.com/blacksmith-community/rabbitmq-forge-boshrelease/issues
[broker]: https://github.com/cloudfoundry-community/blacksmith
[rabbitmq]:  https://rabbitmq.com
