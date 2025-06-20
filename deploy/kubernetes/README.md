# Deployment for the ceph-nvme gateway on OpenShift Data Foundation

## Prerequisites

1. create an OpenShift cluser
1. deploy ODF
1. create a StorageCluster

## Deploy the nvmeof gateway

Throughout this document the `oc` command is used. `kubectl` should work as well.

```
$ oc -n openshift-storage create -f gateway-deployment.yaml
deployment.apps/ceph-nvmeof-gateway created
service/ceph-nvmeof-gateway created
configmap/ceph-nvmeof-config created
```

This creates a Deployment with single replica ceph-nvmeof gateway Pod. The Pod
should get in a `Running` state pretty quickly:
```
$ oc -n openshift-storage get pods -l app=ceph-nvmeof-gateway
NAME                                  READY   STATUS    RESTARTS   AGE
ceph-nvmeof-gateway-86c8449c8-swdzm   1/1     Running   0          86s
```

The logs of the gateway can be followed:
```
$ oc -n openshift-storage logs -f ceph-nvmeof-gateway-54d9c5d489-759nv
Defaulted container "nvmeof-gateway" out of: nvmeof-gateway, generate-minimal-ceph-conf (init)
[20-Jun-2025 15:36:58] INFO utils.py:377 (1): Initialize gateway log level to "DEBUG"
[20-Jun-2025 15:36:58] WARNING utils.py:394 (1): Log files are disabled, the log wouldn't be saved to a file
[20-Jun-2025 15:36:58] INFO config.py:86 (1): Using NVMeoF gateway version 1.5.4
[20-Jun-2025 15:36:58] INFO config.py:89 (1): Configured SPDK version 24.09
[20-Jun-2025 15:36:58] INFO config.py:92 (1): Using vstart cluster version based on 19.2.2
[20-Jun-2025 15:36:58] INFO config.py:95 (1): NVMeoF gateway built on: 2025-06-16 09:28:37 UTC
[20-Jun-2025 15:36:58] INFO config.py:98 (1): NVMeoF gateway Git repository: https://github.com/ceph/ceph-nvmeof
[20-Jun-2025 15:36:58] INFO config.py:101 (1): NVMeoF gateway Git branch: tags/1.5.4
[20-Jun-2025 15:36:58] INFO config.py:104 (1): NVMeoF gateway Git commit: ee451f00e86e5efca62f507052f58843645f549b
[20-Jun-2025 15:36:58] INFO config.py:110 (1): SPDK Git repository: https://github.com/ceph/spdk.git
[20-Jun-2025 15:36:58] INFO config.py:113 (1): SPDK Git branch: undefined
[20-Jun-2025 15:36:58] INFO config.py:116 (1): SPDK Git commit: 7458938fac1f9d34e8010db5b3bc5b9b2a2ba1c4
[20-Jun-2025 15:36:58] INFO config.py:65 (1): Using configuration file /config/ceph-nvmeof.conf
[20-Jun-2025 15:36:58] INFO config.py:67 (1): ====================================== Configuration file content ======================================
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #  based on github.com/ceph/ceph-nvmeof:ceph-nvmeof.conf
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #  Copyright (c) 2021 International Business Machines
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #  All rights reserved.
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #  SPDX-License-Identifier: LGPL-3.0-or-later
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #  Authors: anita.shekar@ibm.com, sandy.kaur@ibm.com
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [gateway]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): name = ceph-nvmeof-gateway
[20-Jun-2025 15:36:58] INFO config.py:72 (1): group =
[20-Jun-2025 15:36:58] INFO config.py:72 (1): addr = 0.0.0.0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): port = 5500
[20-Jun-2025 15:36:58] INFO config.py:72 (1): enable_auth = False
[20-Jun-2025 15:36:58] INFO config.py:72 (1): state_update_notify = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): state_update_timeout_in_msec = 2000
[20-Jun-2025 15:36:58] INFO config.py:72 (1): state_update_interval_sec = 5
[20-Jun-2025 15:36:58] INFO config.py:72 (1): break_update_interval_sec = 25
[20-Jun-2025 15:36:58] INFO config.py:72 (1): enable_spdk_discovery_controller = False
[20-Jun-2025 15:36:58] INFO config.py:72 (1): encryption_key = /etc/ceph/encryption.key
[20-Jun-2025 15:36:58] INFO config.py:72 (1): rebalance_period_sec = 7
[20-Jun-2025 15:36:58] INFO config.py:72 (1): max_gws_in_grp = 16
[20-Jun-2025 15:36:58] INFO config.py:72 (1): max_ns_to_change_lb_grp = 8
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #abort_on_errors = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #omap_file_ignore_unlock_errors = False
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #omap_file_lock_on_read = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #omap_file_lock_duration = 20
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #omap_file_lock_retries = 30
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #omap_file_lock_retry_sleep_interval = 1.0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #omap_file_update_reloads = 10
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #enable_prometheus_exporter = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #prometheus_exporter_ssl = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #prometheus_port = 10008
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #prometheus_bdev_pools = rbd
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #prometheus_stats_interval = 10
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #verify_nqns = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #verify_keys = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #verify_listener_ip = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #allowed_consecutive_spdk_ping_failures = 1
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #spdk_ping_interval_in_seconds = 2.0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_hosts_per_namespace = 8
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_namespaces_with_netmask = 1000
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_subsystems = 128
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_hosts = 2048
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_namespaces = 2048
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_namespaces_per_subsystem = 256
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_hosts_per_subsystem = 32
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [gateway-logs]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): log_level=debug
[20-Jun-2025 15:36:58] INFO config.py:72 (1): log_files_enabled = False
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #log_files_rotation_enabled = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #verbose_log_messages = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_log_file_size_in_mb=10
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_log_files_count=20
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #max_log_directory_backups=10
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Notice that if you change the log directory the log files will only be visible inside the container
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #log_directory = /var/log/ceph/
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [discovery]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): addr = 0.0.0.0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): port = 8009
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #abort_on_errors = True
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [ceph]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): id = admin
[20-Jun-2025 15:36:58] INFO config.py:72 (1): pool = ocs-storagecluster-cephblockpool
[20-Jun-2025 15:36:58] INFO config.py:72 (1): config_file = /etc/ceph/ceph.conf
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [mtls]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): server_key = ./server.key
[20-Jun-2025 15:36:58] INFO config.py:72 (1): client_key = ./client.key
[20-Jun-2025 15:36:58] INFO config.py:72 (1): server_cert = ./server.crt
[20-Jun-2025 15:36:58] INFO config.py:72 (1): client_cert = ./client.crt
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [spdk]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Support multiple cluster allocation strategies
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Legacy strategy, per ANA grp, max bdevs_per_cluster
[20-Jun-2025 15:36:58] INFO config.py:72 (1): bdevs_per_cluster = 32
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Flat bdevs per cluster, ignore ANA grp id
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # flat_bdevs_per_cluster = 32
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Cluster pool
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # cluster_connections = 32
[20-Jun-2025 15:36:58] INFO config.py:72 (1): tgt_path = /usr/local/bin/nvmf_tgt
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #rpc_socket_dir = /var/tmp/
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #rpc_socket_name = spdk.sock
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #tgt_cmd_extra_args = --env-context="--no-huge -m1024" --iova-mode=va
[20-Jun-2025 15:36:58] INFO config.py:72 (1): timeout = 60.0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #log_level =
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #protocol_log_level = WARNING
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #log_file_dir =
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Example value: -m 0x3 -L all
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # tgt_cmd_extra_args =
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # transports = tcp
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Example value: {"max_queue_depth" : 16, "max_io_size" : 4194304, "io_unit_size" : 1048576, "zcopy" : false}
[20-Jun-2025 15:36:58] INFO config.py:72 (1): transport_tcp_options = {"in_capsule_data_size" : 8192, "max_io_qpairs_per_ctrlr" : 7}
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # Example value: {"small_pool_count" : 8192, "large_pool_count" : 1024, "small_bufsize" : 8192, "large_bufsize" : 135168}
[20-Jun-2025 15:36:58] INFO config.py:72 (1): # iobuf_options =
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): qos_timeslice_in_usecs = 0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #notifications_interval = 60
[20-Jun-2025 15:36:58] INFO config.py:72 (1): 
[20-Jun-2025 15:36:58] INFO config.py:72 (1): [monitor]
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #timeout = 1.0
[20-Jun-2025 15:36:58] INFO config.py:72 (1): #log_file_dir =
[20-Jun-2025 15:36:58] INFO config.py:73 (1): ========================================================================================================
[20-Jun-2025 15:36:58] INFO utils.py:377 (1): Initialize gateway log level to "DEBUG"
[20-Jun-2025 15:36:58] WARNING server.py:141 (1): No valid encryption key file was set. Any attempt to encrypt or decrypt keys would fail
[20-Jun-2025 15:36:58] INFO server.py:150 (1): Starting gateway ceph-nvmeof-gateway
[20-Jun-2025 15:36:58] INFO server.py:273 (1): Starting serve, monitor client version: ceph version 20.3.0-913-g1151a9a4 (1151a9a49d0287dba8683a86331aa46f02b61113) tentacle (dev - RelWithDebInfo)
[20-Jun-2025 15:36:58] INFO utils.py:377 (1): Initialize gateway log level to "DEBUG"
2025-06-20T15:36:58.943+0000 7f1a3214f740  1 librados: starting msgr at 
2025-06-20T15:36:58.943+0000 7f1a3214f740  1 librados: starting objecter
2025-06-20T15:36:58.944+0000 7f1a3214f740  1 librados: setting wanted keys
2025-06-20T15:36:58.944+0000 7f1a3214f740  1 librados: calling monclient init
2025-06-20T15:36:58.946+0000 7f1a3214f740  1 librados: init done
2025-06-20T15:36:58.946+0000 7f1a3214f740 10 librados: wait_for_osdmap waiting
2025-06-20T15:36:58.949+0000 7f1a3214f740 10 librados: wait_for_osdmap done waiting
2025-06-20T15:36:58.950+0000 7f1a3214f740 10 librados: create oid=nvmeof.state nspace=
2025-06-20T15:36:58.961+0000 7f1a3214f740 10 librados: Objecter returned from create r=-17
[20-Jun-2025 15:36:58] INFO state.py:770 (1): nvmeof.state OMAP object already exists.
[20-Jun-2025 15:36:58] INFO server.py:400 (1): Starting /usr/bin/ceph-nvmeof-monitor-client --gateway-name ceph-nvmeof-gateway --gateway-address 0.0.0.0:5500 --gateway-pool ocs-storagecluster-cephblockpool --gateway-group  --monitor-group-address 0.0.0.0:5499 -c /etc/ceph/ceph.conf -n client.admin -k /etc/ceph/keyring
[20-Jun-2025 15:36:58] INFO server.py:406 (1): monitor client process id: 18
[20-Jun-2025 15:36:58] INFO server.py:251 (1): MonitorGroup server is listening on 0.0.0.0:5499 for group id
```

A Service is available for connecting to the gateway, the hostname
`ceph-nvmeof-gateway` is assigned to it:
```
$ oc -n openshift-storage get service -l app=ceph-nvmeof-gateway
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ceph-nvmeof-gateway   ClusterIP   172.30.33.186   <none>        5500/TCP,5499/TCP,8009/TCP   5m41s
```

## Using the `nvmeof-cli`

The `oc run` command can create a Pod and start it. Because the `nvmeof-cli`
command exists immediately, the logs of the Pod can be obtained and the Pod
deleted to prepare for a next command.

Check the version of the `quay.io/ceph/nvmeof-cli:latest` container-image:
```
$ oc -n openshift-storage run --image=quay.io/ceph/nvmeof-cli:latest nvmeof-cli -- version
pod/nvmeof-cli created
$ oc -n openshift-storage logs nvmeof-cli
CLI version: 1.5.4
$ oc -n openshift-storage delete pod/nvmeof-cli
pod "nvmeof-cli" deleted
```

Run `namespace list --subsystem SUBSYSTEM_NQN`:
```
$ oc -n openshift-storage run --image=quay.io/ceph/nvmeof-cli:latest nvmeof-cli -- --server-address ceph-nvmeof-gateway --server-port 5500 namespace list
pod/nvmeof-cli created
$ oc -n openshift-storage logs nvmeof-cli
Failure listing namespaces:
<_InactiveRpcError of RPC that terminated with:
	status = StatusCode.UNAVAILABLE
	details = "failed to connect to all addresses; last error: UNKNOWN: ipv4:172.30.33.186:5500: Failed to connect to remote host: Connection refused"
	debug_error_string = "UNKNOWN:failed to connect to all addresses; last error: UNKNOWN: ipv4:172.30.33.186:5500: Failed to connect to remote host: Connection refused {created_time:"2025-06-20T15:40:27.658535316+00:00", grpc_status:14}"
>
$ oc -n openshift-storage delete pod/nvmeof-cli
pod "nvmeof-cli" deleted
```
