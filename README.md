# Blacksmith Kubernetes Forge

This Blacksmith Forge teaches a [Blacksmith Broker][broker] how to
deploy [Kubernetes][kubernetes] clusters, which are all the rage.

## Deploying

To deploy this forge, you will need to add it to your existing
Blacksmith Broker manifest deployment, co-locating the
`kubernetes-blacksmith-plans` job on the Blacksmith instance group.

Here's an example to get you started (clipped for brevity):

```yaml
releases:
  - name:    kubernetes-forge
    version: latest

instance_groups:
  - name: blacksmith
    jobs:
      - name:    kubernetes-blacksmith-plans
        release: kubernetes-forge
        properties:
          plans:
            # your plans here
            # (see below)
```

The Kubernetes Forge deploys kubernetes using the `k8s` BOSH
release available from the [jhunt/k8s-boshrelease][k8s-bosh].
The Forge itself specifes the correct URL and version of the
upstream BOSH release, in the deployment manifests, so unless you
lack access to the internet, you should be ok.

On the other hand, if you do need to upload the release,
Blacksmith is more than happy to upload it to the BOSH director on
your behalf.

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
            - name:    kubernetes
              version: latest
```

Finally, you'll need to define plans for Blacksmith to deploy.
The following sections discuss those in great detail.

## Cluster Topology

Kubernetes only comes in one topology: `cluster`.

### Configuration Options

- **nodes** - How many Kubernetes nodes to spin up.  Defaults to 3.

- **azs** - A list of BOSH availability zones to stripe
  deployments across.  Defaults to [z1,z2,z3].

- **network** - The name of the network to deploy each cluster
  instance to.  Defaults to 'kubernetes-service'.

- **node_cpus** - How many compute cores to allocate each
  Kubernetes node.  Defaults to '2'.

- **node_mem** - How much memory (in megabytes) to allocate each
  Kubernetes node.  Defaults to '2048' (2G).

- **node_ephemeral** - How much ephemeral disk to allocate each
  Kubernetes node.  This must be specified in megabytes.
  Defaults to '20_480' (20G).

- **node_persistent** - How much persistent disk to allocate each
  Kubernetes node.  This must be specified in megabytes.
  Defaults to '20_480' (20G).

### Example Configuration

A single-node cluster plan, with 8G of RAM:

```
instance_groups:
  - name: blacksmith
    jobs:
      - name:    kubernetes-blacksmith-plans
        release: kubernetes-forge
        properties:
          plans:
            lab:
              type:     cluster
              node_mem: 8192
```

Here's a configuration that provides two 5-node cluster plans,
with different node sizing, each in their own networks:

```
instance_groups:
  - name: blacksmith
    jobs:
      - name:    kubernetes-blacksmith-plans
        release: kubernetes-forge
        properties:
          plans:
            internal:
              type:     cluster
              network:  LAN
              nodes:    5
              node_cpu: 2
              node_mem: 4

            external:
              type:     cluster
              network:  WAN
              nodes:    5
              node_cpu: 8
              node_mem: 24
```

## Contributing

If you find a bug, please raise a [Github Issue][1] first,
before submitting a PR.




[1]: https://github.com/blacksmith-community/kubernetes-forge-boshrelease/issues
[broker]: https://github.com/cloudfoundry-community/blacksmith
[kubernetes]:  https://kubernetes.io
