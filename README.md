# NATS Streaming Clustering Helm Chart (alpha)

Sets up a [NATS](http://nats.io/) Streaming server cluster with Raft based replication.

## Getting started

This chart relies on an already available NATS Service to which the
NATS Streaming nodes that will form a clusters can connect to.
You can install the NATS Operator and then use it to create a NATS cluster
via the following:

```console
$ kubectl apply -f https://raw.githubusercontent.com/nats-io/nats-operator/master/example/deployment-rbac.yaml
$ kubectl apply -f https://raw.githubusercontent.com/nats-io/nats-operator/master/example/example-nats-cluster.yaml
```

This will create a NATS cluster on the `nats-io` namespace.  Then, to
install a NATS Streaming cluster the URL to the NATS cluster can be
specified as follows (using `my-release` for a name label for the
cluster):

```console
$ git clone https://github.com/wallyqs/nats-streaming-cluster-chart nats-streaming-cluster
$ helm install nats-streaming-cluster -n my-release  --set natsUrl=nats://nats.nats-io.svc.cluster.local:4222 
```

This will create 3 follower nodes plus an extra Pod which is
configured to be in bootstrapping mode, which will start as the leader
of the Raft group as soon as it joins.

```console
$ kubectl get pods --namespace default -l "app=nats-streaming-cluster,release=my-release"
NAME                                          READY     STATUS    RESTARTS   AGE
my-release-nats-streaming-cluster-0           1/1       Running   0          30s
my-release-nats-streaming-cluster-1           1/1       Running   0          23s
my-release-nats-streaming-cluster-2           1/1       Running   0          17s
my-release-nats-streaming-cluster-bootstrap   1/1       Running   0          30s
```

Note that in case the bootstrapping Pod fails then it will not be
recreated and instead one of the extra follower Pods will take over
the leadership.  The follower Pods are part of a Deployment so those
in case of failure they will be recreated.

## Uninstalling the Chart

To uninstall/delete the my-release deployment:

```console
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the
chart and deletes the release.

## Configuration

| Parameter                                 | Description                                      | Default                                           |
|-------------------------------------------|--------------------------------------------------|---------------------------------------------------|
| `replicaCount`                            | # of NATS streaming servers                      | 3                                                 |
| `image.tag`                               | Container image version                          | 0.9.2                                             |
| `image.pullPolicy`                        | Image pull policy                                | IfNotLatest                                       |
| `clusterID`                               | NATS Streaming Cluster Name ID                   | "test-cluster"                                    |
| `natsUrl`                                 | External NATS Server URL                         | "nats://username:password@nats:4222"              |
| `maxChannels`                             | Max # of channels                                | 100                                               |
| `maxSubs`                                 | Max # of subscriptions per channel               | 1000                                              |
| `maxMsgs`                                 | Max # of messages per channel                    | "1000000"                                         |
| `maxBytes`                                | Max messages total size per channel              | 900mb                                             |
| `maxAge`                                  | Max duration a message can be stored             | "0s" (unlimited)                                  |
| `maxMsgs`                                 | Max # of messages per channel                    | 1000000                                           |
| `debug`                                   | Enable debugging                                 | false                                             |
| `trace`                                   | Enable detailed tracing                          | false  (avoid using this)                         |
| `service.type`                            | ClusterIP, NodePort, LoadBalancer                | ClusterIP                                         |
| `service.monitorPort`                     | Port to accept monitor requests                  | 8222                                              |
| `resources`                               | Server resource requests and limits              | none set                                          |

### File Specific Persistence Configuration

| Parameter                             | Description                                           | Default                                           |
|---------------------------------------|-------------------------------------------------------|---------------------------------------------------|
| `persistence.file.compactEnabled`     | Enable compaction                                     | true                                              |
| `persistence.file.bufferSize`         | File buffer size (in bytes)                           | "2097152"  (2MB)                                  |
| `persistence.file.crc`                | Enable file CRC-32 checksum                           | true                                              | 
| `persistence.file.sync`               | Enable File.Sync on Flush                             | true                                              |
| `persistence.file.fdsLimit`           | Max File Descriptor limit (approx)                    | 0 (unlimited)                                     |

*Additional configuration parameters not typically used may be found in [values.yaml](values.yaml).*
