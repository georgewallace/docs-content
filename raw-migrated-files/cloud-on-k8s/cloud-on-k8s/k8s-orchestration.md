# Nodes orchestration [k8s-orchestration]

This section covers the following topics:

* [NodeSets overview](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-nodesets)
* [Cluster upgrade](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-upgrading)
* [Cluster upgrade patterns](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-upgrade-patterns)
* [StatefulSets orchestration](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-statefulsets)
* [Limitations](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-orchestration-limitations)

## NodeSets overview [k8s-nodesets]

NodeSets are used to specify the topology of the Elasticsearch cluster. Each NodeSet represents a group of Elasticsearch nodes that share the same Elasticsearch configuration and Kubernetes Pod configuration.

::::{tip}
You can use [YAML anchors](https://yaml.org/spec/1.2/spec.md#id2765878) to declare the configuration change once and reuse it across all the node sets.
::::


```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 8.16.1
  nodeSets:
  - name: master-nodes
    count: 3
    config:
      node.roles: ["master"]
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi
        storageClassName: standard
  - name: data-nodes
    count: 10
    config:
      node.roles: ["data"]
    volumeClaimTemplates:
    - metadata:
        name: elasticsearch-data
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1000Gi
        storageClassName: standard
```

In this example, the Elasticsearch resource defines two NodeSets:

* `master-nodes` with 10Gi volumes
* `data-nodes` with 1000Gi volumes

The Elasticsearch cluster is composed of 13 nodes: 3 master nodes and 10 data nodes.


## Upgrading the cluster [k8s-upgrading]

ECK handles smooth upgrades from one cluster specification to another. You can apply a new Elasticsearch specification at any time. For example, based on the Elasticsearch specification described in the [NodeSets overview](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-nodesets), you can:

* Add five additional Elasticsearch data nodes: In `data-nodes` change the value in the `count` field from 10 to 15.
* Increase the memory limit of data nodes to 32Gi: Set a [different resource limit](../../../deploy-manage/deploy/cloud-on-k8s/manage-compute-resources.md) in the existing `data-nodes` NodeSet.
* Replace dedicated master and dedicated data nodes with nodes having both master and data roles: Replace the two existing NodeSets by a single one with a different name and the appropriate Elasticsearch configuration settings.
* Upgrade Elasticsearch from version 7.2.0 to 7.3.0: Change the value in the `version` field.

ECK orchestrates NodeSet changes with no downtime and makes sure that:

* Before a node is removed, the relevant data is migrated to other nodes (with some [limitations](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-orchestration-limitations)).
* When a cluster topology changes, these Elasticsearch settings are adjusted accordingly:

    * `discovery.seed_hosts`
    * `cluster.initial_master_nodes`
    * `discovery.zen.minimum_master_nodes`
    * `_cluster/voting_config_exclusions`

* Rolling upgrades are performed safely with existing PersistentVolumes reused where possible.


## StatefulSets orchestration [k8s-statefulsets]

Behind the scenes, ECK translates each NodeSet specified in the Elasticsearch resource into a [StatefulSet in Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/). The NodeSet specification is based on the StatefulSet specification:

* `count` corresponds to the number of replicas in the StatefulSet. A StatefulSet replica is a Pod — which corresponds to an Elasticsearch node.
* `podTemplate` can be used to [customize some aspects of the Elasticsearch Pods](../../../deploy-manage/deploy/cloud-on-k8s/customize-pods.md) created by the underlying StatefulSet.
* The StatefulSet name is derived from the Elasticsearch resource name and the NodeSet name. Each Pod in the StatefulSet gets a name generated by suffixing the pod ordinal to the StatefulSet name. Elasticsearch nodes have the same name as the Pod they are running on.

The actual Pod creation is handled by the StatefulSet controller in Kubernetes. ECK relies on the [OnDelete StatefulSet update strategy](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#update-strategies) since it needs full control over when and how Pods get upgraded to a new revision.

When a Pod is removed and recreated (maybe with a newer revision), the StatefulSet controller makes sure that the PersistentVolumes attached to the original Pod are then attached to the new Pod.


## Cluster upgrade patterns [k8s-upgrade-patterns]

Depending on how the NodeSets are updated, ECK handles the Kubernetes resource reconciliation in various ways.

* A new NodeSet is added to the Elasticsearch resource.

    ECK creates the corresponding StatefulSet. It also sets up [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) and [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) to hold the TLS certificates and Elasticsearch configuration files.

* The node count of an existing NodeSet is increased.

    ECK increases the replicas of the corresponding StatefulSet.

* The node count of an existing NodeSet is decreased.

    ECK migrates data away from the Elasticsearch nodes due to be removed and then decreases the replicas of the corresponding StatefulSet. [PersistentVolumeClaims](../../../deploy-manage/deploy/cloud-on-k8s/volume-claim-templates.md) belonging to the removed nodes are automatically removed as well.

* An existing NodeSet is removed.

    ECK migrates data away from the Elasticsearch nodes in the NodeSet and removes the underlying StatefulSet.

* The specification of an existing NodeSet is updated. For example, the Elasticsearch configuration, or the PodTemplate resources requirements.

    ECK performs a rolling upgrade of the corresponding Elasticsearch nodes. It follows the [Elasticsearch rolling upgrade best practices](/deploy-manage/upgrade/deployment-or-cluster.md) to update the underlying Pods while maintaining the availability of the Elasticsearch cluster where possible. In most cases, the process simply involves restarting Elasticsearch nodes one-by-one. Note that some cluster topologies may be impossible to deploy without making the cluster unavailable (check [Limitations](../../../deploy-manage/upgrade/deployment-or-cluster.md#k8s-orchestration-limitations) ).

* An existing NodeSet is renamed.

    ECK creates a new NodeSet with the new name, migrates data away from the old NodeSet, and then removes it. During this process the Elasticsearch cluster could temporarily have more nodes than normal. The Elasticsearch [update strategy](../../../deploy-manage/deploy/cloud-on-k8s/update-strategy.md) controls how many nodes can exist above or below the target node count during the upgrade.


In all these cases, ECK handles StatefulSet operations according to the Elasticsearch orchestration best practices by adjusting the following orchestration settings:

* `discovery.seed_hosts`
* `cluster.initial_master_nodes`
* `discovery.zen.minimum_master_nodes`
* `_cluster/voting_config_exclusions`


## Limitations [k8s-orchestration-limitations]

Due to relying on Kubernetes primitives such as StatefulSets, the ECK orchestration process has some inherent limitations.

* Cluster availability during rolling upgrades is not guaranteed for the following cases:

    * Single-node clusters
    * Clusters containing indices with no replicas


If an {{es}} node holds the only copy of a shard, this shard becomes unavailable while the node is upgraded. To ensure [high availability](/deploy-manage/production-guidance/availability-and-resilience.md) it is recommended to configure clusters with three master nodes, more than one node per [data tier](/manage-data/lifecycle/data-tiers.md) and at least one replica per index.

* Elasticsearch Pods may stay `Pending` during a rolling upgrade if the Kubernetes scheduler cannot re-schedule them back. This is especially important when using local PersistentVolumes. If the Kubernetes node bound to a local PersistentVolume does not have enough capacity to host an upgraded Pod which was temporarily removed, that Pod will stay `Pending`.
* Rolling upgrades can only make progress if the Elasticsearch cluster health is green. There are exceptions to this rule if the cluster health is yellow and if the following conditions are satisfied:

    * A cluster version upgrade is in progress and some Pods are not up to date.
    * There are no initializing or relocating shards.


If these conditions are met, then ECK can delete a Pod for upgrade even if the cluster health is yellow, as long as the Pod is not holding the last available replica of a shard.

The health of the cluster is deliberately ignored in the following cases:

* If all the Elasticsearch nodes of a NodeSet are unavailable, probably caused by a misconfiguration, the operator ignores the cluster health and upgrades nodes of the NodeSet.
* If an Elasticsearch node to upgrade is not healthy, and not part of the Elasticsearch cluster, the operator ignores the cluster health and upgrades the Elasticsearch node.

    * Elasticsearch versions cannot be downgraded. For example, it is impossible to downgrade an existing cluster from version 7.3.0 to 7.2.0. This is not supported by Elasticsearch.


Advanced users may force an upgrade by manually deleting Pods themselves. The deleted Pods are automatically recreated at the latest revision.

Operations that reduce the number of nodes in the cluster cannot make progress without user intervention, if the Elasticsearch index replica settings are incompatible with the intended downscale. Specifically, if the Elasticsearch index settings demand a higher number of shard copies than data nodes in the cluster after the downscale operation, ECK cannot migrate the data away from the node about to be removed. You can address this in the following ways:

* Adjust the Elasticsearch [index settings](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-put-settings) to a number of replicas that allow the desired node removal.
* Use [`auto_expand_replicas`](asciidocalypse://docs/elasticsearch/docs/reference/elasticsearch/index-settings/index.md#dynamic-index-settings) to automatically adjust the replicas to the number of data nodes in the cluster.


## Advanced control during rolling upgrades [k8s-advanced-upgrade-control]

The rules (otherwise known as predicates) that the ECK operator follows during an Elasticsearch upgrade can be selectively disabled for certain scenarios where the ECK operator will not proceed with an Elasticsearch cluster upgrade because it deems it to be "unsafe".

::::{warning}
Selectively disabling the predicates listed in this section are extremely risky, and carry a high chance of either data loss, or causing a cluster to become completely unavailable. Use them only if you are sure that you are not causing permanent damage to an Elasticsearch cluster. These predicates might change in the future. We will be adding, removing, and renaming these over time, so be careful in adding these to any automation.  Also, make sure you remove them after use. `kublectl annotate elasticsearch.elasticsearch.k8s.elastic.co/elasticsearch-sample eck.k8s.elastic.co/disable-upgrade-predicates-`
::::


* The following named predicates control the upgrade process

    * data_tier_with_higher_priority_must_be_upgraded_first

        Upgrade the frozen tier first, then the cold tier, then the warm tier, and the hot tier last. This ensures ILM can continue to move data through the tiers during the upgrade.

    * do_not_restart_healthy_node_if_MaxUnavailable_reached

        If `maxUnavailable` is reached, only allow unhealthy Pods to be deleted.

    * skip_already_terminating_pods

        Do not attempt to restart pods that are already in the process of being terminated.

    * only_restart_healthy_node_if_green_or_yellow

        Only restart healthy Elasticsearch nodes if the health of the cluster is either green or yellow, never red.

    * if_yellow_only_restart_upgrading_nodes_with_unassigned_replicas

        During a rolling upgrade, primary shards assigned to a node running a new version cannot have their replicas assigned to a node with the old version. Therefore we must allow some Pods to be restarted even if cluster health is yellow so the replicas can be assigned.

    * require_started_replica

        If a cluster is yellow, allow deleting a node, but only if they do not contain the only replica of a shard since it would make the cluster go red.

    * one_master_at_a_time

        Only allow a single master to be upgraded at a time.

    * do_not_delete_last_master_if_all_master_ineligible_nodes_are_not_upgraded

        Force an upgrade of all the master-ineligible nodes before upgrading the last master-eligible node.

    * do_not_delete_pods_with_same_shards

        Do not allow two pods containing the same shard to be deleted at the same time.

    * do_not_delete_all_members_of_a_tier

        Do not delete all nodes that share the same node roles at once. This ensures that there is always availability for each configured tier of nodes during a rolling upgrade.


Any of these predicates can be disabled by adding an annotation with the key of `eck.k8s.elastic.co/disable-upgrade-predicates` to the Elasticsearch metadata, specifically naming the predicate that is needing to be disabled.  Also, all predicates can be disabled by replacing the name of any predicatae with "*".

* Example use case

Assume a given Elasticsearch cluster is a "red" state because of an un-allocatable shard setting that was applied to the cluster:

```json
{
	"settings": {
		"index.routing.allocation.include._id": "does not exist"
	}
}
```

This cluster would never be allowed to be upgraded with the standard set of upgrade predicates in place, as the cluster is in a "red" state, and the named predicate `only_restart_healthy_node_if_green_or_yellow` prevents the upgrade.

If the following annotation was added to the cluster specification, and the version was increased from 7.15.2 → 7.15.3

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: testing
  annotations:
    eck.k8s.elastic.co/disable-upgrade-predicates: "only_restart_healthy_node_if_green_or_yellow"
    # Also note that eck.k8s.elastic.co/disable-upgrade-predicates: "*" would work as well, but is much less selective.
spec:
  version: 7.15.3 # previously set to 7.15.2, for example
```

The ECK operator would allow this upgrade to proceed, even though the cluster was in a "red" state during this upgrade process.


