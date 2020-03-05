---
title: In-Memory Storage and Correctness
description: How Jet makes use of Hazelcast's in-memory storage
---

A distinctive feature of Hazelcast Jet is that it has no dependency on
disk storage, it keeps all of its operational state in the RAM of the
cluster. It also doesn't delegate its fault tolerance concerns to an
outside system like ZooKeeper. This comes from the fact that it's built
on top of Hazelcast IMDG.

## Level of Safety

Jet backs up the state to its own `IMap` objects. `IMap` is a replicated
in-memory data structure, storing each key-value pair on a configurable
number of cluster members. By default it makes a single backup copy,
resulting in a system that tolerates the failure of a single member at a
time. The cluster recovers its safety level by re-establishing all the
missing backups, and when this is done, another node can fail without
data loss. You can tweak the backup count in the configuration, for
example:

```java
JetConfig config = new JetConfig();
config.getInstanceConfig().setBackupCount(2);
JetInstance instance = Jet.newJetInstance(config);
```

If multiple members fail simultaneously, some data from the backing
`IMap`s can be lost. Jet detects this by counting the entries in the
snapshot `IMap`, so it won't run a job  with missing data.