# Deployment for the ceph-nvmeof gateway on OpenShift Data Foundation

## Prerequisites

1. create an OpenShift cluser
1. deploy ODF
1. create a StorageCluster

## Deploy the nvmeof gateway

Throughout this document the `oc` command is used. `kubectl` should work as well.

```
$ oc -n openshift-storage create -f gateway-deployment.yaml
serviceaccount/ceph-nvmeof created
securitycontextconstraints.security.openshift.io/ceph-nvmeof created
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
...
```

A Service is available for connecting to the gateway, the hostname
`ceph-nvmeof-gateway` is assigned to it:
```
$ oc -n openshift-storage get service -l app=ceph-nvmeof-gateway
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ceph-nvmeof-gateway   ClusterIP   172.30.33.186   <none>        5500/TCP,5499/TCP,8009/TCP   5m41s
```

## Using the `nvmeof-cli`

A simple toolbox Pod with `nvmeof-cli` can be deployed with:

```
$ oc -n openshift-storage create -f nvmeof-cli-toolbox.yaml
pod/nvmeof-cli created
```

In order to use the `nvmeof-cli`, it is need to remote shell into the running Pod:

```
$ oc -n openshift-storage rsh nvmeof-cli
sh-5.1# nvmeof-cli version
CLI version: 1.5.7
sh-5.1# nvmeof-cli --help
usage: python3 -m control.cli [-h] [--format {text,json,yaml,plain,python}] [--output {log,stdio}] [--log-level {debug,DEBUG,info,INFO,warning,WARNING,error,ERROR,critical,CRITICAL}]
                              [--server-address SERVER_ADDRESS] [--server-port SERVER_PORT] [--client-key CLIENT_KEY] [--client-cert CLIENT_CERT] [--server-cert SERVER_CERT] [--verbose]
                              {version,gateway,gw,spdk_log_level,subsystem,listener,host,connection,namespace,ns,get_subsystems} ...

CLI to manage NVMe gateways

optional arguments:
  -h, --help            show this help message and exit
  --format {text,json,yaml,plain,python}
                        CLI output format
  --output {log,stdio}  CLI output method
  --log-level {debug,DEBUG,info,INFO,warning,WARNING,error,ERROR,critical,CRITICAL}
                        CLI log level
  --server-address SERVER_ADDRESS
                        Server address (default: CEPH_NVMEOF_SERVER_ADDRESS env variable or 'localhost')
  --server-port SERVER_PORT
                        Server port (default: CEPH_NVMEOF_SERVER_PORT env variable or '5500')
  --client-key CLIENT_KEY
                        Path to the client key file
  --client-cert CLIENT_CERT
                        Path to the client certificate file
  --server-cert SERVER_CERT
                        Path to the server certificate file
  --verbose             Run CLI in verbose mode

Commands:
  {version,gateway,gw,spdk_log_level,subsystem,listener,host,connection,namespace,ns,get_subsystems}
    version             Get CLI version
    gateway (gw)        Gateway commands
    spdk_log_level      SPDK log level commands
    subsystem           Subsystem commands
    listener            Listener commands
    host                Host commands
    connection          Connection commands
    namespace (ns)      Namespace commands
    get_subsystems      Get subsystems
sh-5.1# 
```


When the Toolbox Pod is not needed anymore, it can be deleted:

```
$ oc -n openshift-storage delete pod/nvmeof-cli
pod "nvmeof-cli" deleted
```


## Current Status

Most likely there are configuration errors.

- trying to create a NVMe-oF Namespace with Ceph-CSI as [described
  elsewhere](https://github.com/nixpanic/ceph-csi/blob/nvmeof/testing/STATUS-NVMe-oF.md)
- fails with `GWs not found`, [needs
  Tentacle](https://github.com/ceph/ceph-nvmeof/issues/1382)?

```
$ oc -n openshift-storage logs ceph-nvmeof-gateway-5fc7679485-txntq
Defaulted container "nvmeof-gateway" out of: nvmeof-gateway, generate-minimal-ceph-conf (init)
[01-Aug-2025 14:18:40] INFO utils.py:380 (1): Initialize gateway log level to "DEBUG"
[01-Aug-2025 14:18:40] INFO utils.py:395 (1): Log files will be saved in /var/log/ceph/nvmeof-ceph-nvmeof-gateway-5fc7679485-txntq, using rotation
[01-Aug-2025 14:18:40] INFO config.py:86 (1): Using NVMeoF gateway version 1.5.7
[01-Aug-2025 14:18:40] INFO config.py:89 (1): Configured SPDK version 25.05
[01-Aug-2025 14:18:40] INFO config.py:92 (1): Using vstart cluster version based on 19.2.2
[01-Aug-2025 14:18:40] INFO config.py:95 (1): NVMeoF gateway built on: 2025-07-31 14:31:00 UTC
[01-Aug-2025 14:18:40] INFO config.py:98 (1): NVMeoF gateway Git repository: https://github.com/ceph/ceph-nvmeof
[01-Aug-2025 14:18:40] INFO config.py:101 (1): NVMeoF gateway Git branch: devel
[01-Aug-2025 14:18:40] INFO config.py:104 (1): NVMeoF gateway Git commit: f25a398cd79d01e5fa638780f6bce566eef986cf
[01-Aug-2025 14:18:40] INFO config.py:110 (1): SPDK Git repository: https://github.com/ceph/spdk.git
[01-Aug-2025 14:18:40] INFO config.py:113 (1): SPDK Git branch: undefined
[01-Aug-2025 14:18:40] INFO config.py:116 (1): SPDK Git commit: 11e9475509a0eef77ececc6ed27714b64644cbcb
[01-Aug-2025 14:18:40] INFO config.py:65 (1): Using configuration file /etc/ceph/nvmeof.conf
[01-Aug-2025 14:18:40] INFO config.py:67 (1): ====================================== Configuration file content ======================================
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # heavily based on tests/ceph-nvmeof.no-huge.conf
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # Changes are annotated with "TODO:".
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #  Copyright (c) 2021 International Business Machines
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #  All rights reserved.
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #  SPDX-License-Identifier: LGPL-3.0-or-later
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #  Authors: anita.shekar@ibm.com, sandy.kaur@ibm.com
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [gateway]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): name = ceph-nvmeof-gateway-5fc7679485-txntq
[01-Aug-2025 14:18:40] INFO config.py:72 (1): group = openshift-storage
[01-Aug-2025 14:18:40] INFO config.py:72 (1): addr = 10.129.2.49
[01-Aug-2025 14:18:40] INFO config.py:72 (1): port = 5500
[01-Aug-2025 14:18:40] INFO config.py:72 (1): enable_auth = False
[01-Aug-2025 14:18:40] INFO config.py:72 (1): state_update_notify = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): state_update_timeout_in_msec = 2000
[01-Aug-2025 14:18:40] INFO config.py:72 (1): state_update_interval_sec = 5
[01-Aug-2025 14:18:40] INFO config.py:72 (1): enable_spdk_discovery_controller = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): encryption_key = /etc/ceph/encryption.key
[01-Aug-2025 14:18:40] INFO config.py:72 (1): rebalance_period_sec = 7
[01-Aug-2025 14:18:40] INFO config.py:72 (1): max_gws_in_grp = 16
[01-Aug-2025 14:18:40] INFO config.py:72 (1): max_ns_to_change_lb_grp = 8
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #omap_file_lock_duration = 20
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #omap_file_lock_retries = 30
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #omap_file_lock_retry_sleep_interval = 1.0
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #omap_file_update_reloads = 10
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #enable_prometheus_exporter = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #prometheus_exporter_ssl = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #prometheus_port = 10008
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #prometheus_bdev_pools = rbd
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #prometheus_stats_interval = 10
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #verify_nqns = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #verify_keys = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #verify_listener_ip = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #allowed_consecutive_spdk_ping_failures = 1
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #spdk_ping_interval_in_seconds = 2.0
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_hosts_per_namespace = 8
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_namespaces_with_netmask = 1000
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_subsystems = 128
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_hosts = 2048
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_namespaces = 2048
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_namespaces_per_subsystem = 256
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_hosts_per_subsystem = 128
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # TODO: disabled monitor client, otherwise hangs at
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # "MonitorGroup server is listening on 0.0.0.0:5499 for group id"
[01-Aug-2025 14:18:40] INFO config.py:72 (1): enable_monitor_client = False
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [gateway-logs]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): log_level=debug
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #log_files_enabled = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #log_files_rotation_enabled = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #verbose_log_messages = True
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_log_file_size_in_mb=10
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_log_files_count=20
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #max_log_directory_backups=10
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # Notice that if you change the log directory the log files will only be visible inside the container
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #log_directory = /var/log/ceph/
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [discovery]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): addr = 10.129.2.49
[01-Aug-2025 14:18:40] INFO config.py:72 (1): port = 8009
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [ceph]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # TODO: added "id" and updated "pool" for ODF
[01-Aug-2025 14:18:40] INFO config.py:72 (1): id = admin
[01-Aug-2025 14:18:40] INFO config.py:72 (1): pool = ocs-storagecluster-cephblockpool
[01-Aug-2025 14:18:40] INFO config.py:72 (1): config_file = /etc/ceph/ceph.conf
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [mtls]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): server_key = ./server.key
[01-Aug-2025 14:18:40] INFO config.py:72 (1): client_key = ./client.key
[01-Aug-2025 14:18:40] INFO config.py:72 (1): server_cert = ./server.crt
[01-Aug-2025 14:18:40] INFO config.py:72 (1): client_cert = ./client.crt
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [spdk]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): bdevs_per_cluster = 32
[01-Aug-2025 14:18:40] INFO config.py:72 (1): mem_size=4096
[01-Aug-2025 14:18:40] INFO config.py:72 (1): tgt_path = /usr/local/bin/nvmf_tgt
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #rpc_socket_dir = /var/tmp/
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #rpc_socket_name = spdk.sock
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #tgt_cmd_extra_args = --env-context="--no-huge -m1024" --iova-mode=va
[01-Aug-2025 14:18:40] INFO config.py:72 (1): timeout = 60.0
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #log_level =
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #protocol_log_level = WARNING
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #log_file_dir =
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # Example value: -m 0x3 -L all
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # tgt_cmd_extra_args =
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # transports = tcp
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # Example value: {"max_queue_depth" : 16, "max_io_size" : 4194304, "io_unit_size" : 1048576, "zcopy" : false}
[01-Aug-2025 14:18:40] INFO config.py:72 (1): transport_tcp_options = {"in_capsule_data_size" : 8192, "max_io_qpairs_per_ctrlr" : 7}
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # Example value: {"small_pool_count" : 8192, "large_pool_count" : 1024, "small_bufsize" : 8192, "large_bufsize" : 135168}
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # iobuf_options =
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): # qos_timeslice_in_usecs = 0
[01-Aug-2025 14:18:40] INFO config.py:72 (1): 
[01-Aug-2025 14:18:40] INFO config.py:72 (1): [monitor]
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #timeout = 1.0
[01-Aug-2025 14:18:40] INFO config.py:72 (1): #log_file_dir =
[01-Aug-2025 14:18:40] INFO config.py:73 (1): ========================================================================================================
[01-Aug-2025 14:18:40] WARNING server.py:144 (1): No valid encryption key file was set. Any attempt to encrypt or decrypt keys would fail
[01-Aug-2025 14:18:40] INFO server.py:153 (1): Starting gateway ceph-nvmeof-gateway-5fc7679485-txntq
[01-Aug-2025 14:18:40] INFO server.py:287 (1): Starting serve, monitor client version: ceph version 20.3.0-1996-gf5aafe05 (f5aafe056e3a81969333ae5b05a6936bf225f2f4) tentacle (dev - RelWithDebInfo)
[01-Aug-2025 14:18:40] INFO state.py:770 (1): nvmeof.openshift-storage.state OMAP object already exists.
[01-Aug-2025 14:18:40] WARNING utils.py:765 (1): No matching DSA devices found (8086:0b25).
[01-Aug-2025 14:18:40] INFO server.py:383 (1): CEPH monitor client is disabled
[01-Aug-2025 14:18:40] DEBUG server.py:562 (1): Configuring server ceph-nvmeof-gateway-5fc7679485-txntq
[01-Aug-2025 14:18:40] INFO server.py:564 (1): SPDK Target Path: /usr/local/bin/nvmf_tgt
[01-Aug-2025 14:18:40] INFO server.py:581 (1): SPDK Socket: /var/tmp/spdk.sock
[01-Aug-2025 14:18:40] INFO server.py:597 (1): SPDK will not use huge pages, mem size: 4096
[01-Aug-2025 14:18:40] INFO server.py:607 (1): SPDK autodetecting cpu_mask: -m 0xF
[01-Aug-2025 14:18:40] INFO server.py:621 (1): Starting /usr/local/bin/nvmf_tgt --wait-for-rpc -u -r /var/tmp/spdk.sock --no-huge -s 4096 -m 0xF
[01-Aug-2025 14:18:42] INFO server.py:644 (1): SPDK process id: 18
[01-Aug-2025 14:18:42] INFO server.py:647 (1): Attempting to initialize SPDK: rpc_socket: /var/tmp/spdk.sock, conn_retries: 300, timeout: 60.0, log level: WARNING
INFO: Setting log level to WARNING
[01-Aug-2025 14:18:42] INFO client.py:110 (1): Setting log level to WARNING
[2025-08-01 14:18:42.278554] Starting SPDK v25.05 / DPDK 25.03.0 initialization...
[2025-08-01 14:18:42.278663] [ DPDK EAL parameters: nvmf --no-shconf -c 0xF -m 4096 --no-pci --no-huge --legacy-mem --iova-mode=va --no-telemetry --log-level=lib.eal:6 --log-level=lib.cryptodev:5 --log-level=lib.power:5 --log-level=user1:6 --base-virtaddr=0x200000000000 --file-prefix=spdk_pid18 ]
[2025-08-01 14:18:42.360934] app.c: 919:spdk_app_start: *NOTICE*: Total cores available: 4
[2025-08-01 14:18:42.429655] reactor.c: 996:reactor_run: *NOTICE*: Reactor started on core 1
[2025-08-01 14:18:42.429699] reactor.c: 996:reactor_run: *NOTICE*: Reactor started on core 2
[2025-08-01 14:18:42.429761] reactor.c: 996:reactor_run: *NOTICE*: Reactor started on core 3
[2025-08-01 14:18:42.429770] reactor.c: 996:reactor_run: *NOTICE*: Reactor started on core 0
[2025-08-01 14:18:42.466455] accel_dsa_rpc.c:  48:rpc_dsa_scan_accel_module: *NOTICE*: Enabled DSA kernel-mode
[01-Aug-2025 14:18:42] DEBUG server.py:867 (1): dsa_scan_accel_module: res=True
[2025-08-01 14:18:42.475104] accel.c:3146:accel_module_initialize: *NOTICE*: No devices for module dsa, skipping
[01-Aug-2025 14:18:42] DEBUG server.py:881 (1): create_transport: tcp options: {"in_capsule_data_size" : 8192, "max_io_qpairs_per_ctrlr" : 7}
[2025-08-01 14:18:42.586765] tcp.c: 759:nvmf_tcp_create: *NOTICE*: *** TCP Transport Init ***
[01-Aug-2025 14:18:42] INFO server.py:707 (1): Started SPDK with version "SPDK v25.05"
[01-Aug-2025 14:18:42] INFO server.py:439 (1): Using SPDK discovery service
[01-Aug-2025 14:18:42] DEBUG state.py:320 (1): Init OMAP lock, cookie: omap_file_cookie_0x7f1b21241610_1_1_exclusive, thread: 1, self: 0x7f1b21241610
[01-Aug-2025 14:18:42] DEBUG cephutils.py:39 (1): Execute monitor command: {"prefix":"mon versions"}
[01-Aug-2025 14:18:42] DEBUG cephutils.py:42 (1): Monitor reply: (0, b'{\n    "ceph version 19.2.1-222.el9cp (f2cd71cc2f7b46709c2351134ac89ea3e9f609b6) squid (stable)": 3\n}\n', '')
[01-Aug-2025 14:18:42] INFO cephutils.py:124 (1): Connected to Ceph with version "19.2.1-222.el9cp (f2cd71cc2f7b46709c2351134ac89ea3e9f609b6) squid (stable)"
[01-Aug-2025 14:18:42] INFO cluster.py:68 (1): NVMeoF bdevs per cluster: 32
[01-Aug-2025 14:18:42] INFO rebalance.py:35 (1):  Starting rebalance thread: period: 7, max number ns to move: 8
[01-Aug-2025 14:18:42] DEBUG rebalance.py:43 (1): Rebalance thread id is 38
[01-Aug-2025 14:18:42] DEBUG cephutils.py:82 (1): nvme-show string: {"prefix":"nvme-gw show", "pool":"ocs-storagecluster-cephblockpool", "group":"openshift-storage"}
[01-Aug-2025 14:18:42] DEBUG cephutils.py:39 (1): Execute monitor command: {"prefix":"nvme-gw show", "pool":"ocs-storagecluster-cephblockpool", "group":"openshift-storage"}
2025-08-01T14:18:42.665+0000 7f1b253b6e00  1 mgrc service_daemon_register nvmeof.ceph-nvmeof-gateway-5fc7679485-txntq metadata {arch=x86_64,ceph_release=tentacle,ceph_version=ceph version 20.3.0-1996-gf5aafe05 (f5aafe056e3a81969333ae5b05a6936bf225f2f4) tentacle (dev - RelWithDebInfo),ceph_version_short=20.3.0-1996-gf5aafe05,cpu=Intel(R) Xeon(R) CPU E5-2686 v4 @ 2.30GHz,daemon_type=gateway,distro=rhel,distro_description=Red Hat Enterprise Linux 9.6 (Plow),distro_version=9.6,group=openshift-storage,hostname=ceph-nvmeof-gateway-5fc7679485-txntq,id=ceph-nvmeof-gateway-5fc7679485-txntq,kernel_description=#1 SMP PREEMPT_DYNAMIC Fri Apr 4 10:41:31 EDT 2025,kernel_version=5.14.0-570.12.1.el9_6.x86_64,mem_swap_kb=0,mem_total_kb=65811756,os=Linux,pool_name=ocs-storagecluster-cephblockpool}
[01-Aug-2025 14:18:42] DEBUG cephutils.py:42 (1): Monitor reply: (0, b'{\n    "epoch": 1,\n    "pool": "ocs-storagecluster-cephblockpool",\n    "group": "openshift-storage",\n    "features": "LB",\n    "num gws": 0\n}\n', '')
[01-Aug-2025 14:18:42] INFO cephutils.py:147 (1): Registered ceph-nvmeof-gateway-5fc7679485-txntq to service_map!
[01-Aug-2025 14:18:42] INFO server.py:268 (1): Prometheus exporter endpoint is enabled
[01-Aug-2025 14:18:42] DEBUG state.py:476 (1): Locked OMAP shared, thread id: 75, id: gateway-ceph-nvmeof-gateway-5fc7679485-txntq, cookie: omap_file_cookie_0x7f1b21241610_1_75_1754057922664814042_shared
[01-Aug-2025 14:18:42] DEBUG cephutils.py:84 (1): reply "(0, b'{\n    "epoch": 1,\n    "pool": "ocs-storagecluster-cephblockpool",\n    "group": "openshift-storage",\n    "features": "LB",\n    "num gws": 0\n}\n', '')"
[01-Aug-2025 14:18:42] INFO server.py:273 (1): Delaying Prometheus exporter startup by 240 seconds...
[01-Aug-2025 14:18:42] DEBUG state.py:855 (1): Locked OMAP file before reading its content (gateway-ceph-nvmeof-gateway-5fc7679485-txntq)
[01-Aug-2025 14:18:42] INFO server.py:280 (1): Prometheus exporter startup scheduled in background
[01-Aug-2025 14:18:42] INFO cephutils.py:92 (1): illegal rebalance ana group  0
[01-Aug-2025 14:18:42] DEBUG cephutils.py:95 (1): Rebalance ana_group: 0, num-gws: 0
[01-Aug-2025 14:18:42] WARNING cephutils.py:110 (1): GWs not found
[01-Aug-2025 14:18:42] DEBUG rebalance.py:114 (1): Called rebalance logic: current rebalancing ana group 0
[01-Aug-2025 14:18:42] INFO rebalance.py:117 (1): Auto rebalance is not supported - index 0
[01-Aug-2025 14:18:42] DEBUG rebalance.py:52 (1): Nothing found for rebalance, break at 0 iteration
[01-Aug-2025 14:18:42] DEBUG state.py:577 (1): OMAP was unlocked, thread id: 75, id: gateway-ceph-nvmeof-gateway-5fc7679485-txntq, cookie: omap_file_cookie_0x7f1b21241610_1_75_1754057922664814042_shared
[01-Aug-2025 14:18:42] DEBUG state.py:862 (1): Released OMAP file lock after reading content (gateway-ceph-nvmeof-gateway-5fc7679485-txntq)
[01-Aug-2025 14:18:42] DEBUG state.py:1413 (1): Check local version 1 against OMAP version 2 (gateway-ceph-nvmeof-gateway-5fc7679485-txntq).
[01-Aug-2025 14:18:42] INFO state.py:1416 (1): Start update from 1 to 2 (gateway-ceph-nvmeof-gateway-5fc7679485-txntq).
[01-Aug-2025 14:18:42] DEBUG state.py:1424 (1): Added keys: {'subsystem_nqn.2016-06.io.spdk:cnode1', 'omap_version'}
[01-Aug-2025 14:18:42] INFO grpc.py:1545 (1): Received request to create subsystem nqn.2016-06.io.spdk:cnode1, enable_ha: True, max_namespaces: 512, no group append: True, context: None
[01-Aug-2025 14:18:42] DEBUG grpc.py:1690 (1): create_subsystem nqn.2016-06.io.spdk:cnode1: True
[01-Aug-2025 14:18:42] INFO state.py:1663 (1): Update complete (1 -> 2) (gateway-ceph-nvmeof-gateway-5fc7679485-txntq).
[01-Aug-2025 14:18:47] DEBUG state.py:476 (1): Locked OMAP shared, thread id: 75, id: gateway-ceph-nvmeof-gateway-5fc7679485-txntq, cookie: omap_file_cookie_0x7f1b21241610_1_75_1754057927664901697_shared
[01-Aug-2025 14:18:47] DEBUG state.py:855 (1): Locked OMAP file before reading its content (gateway-ceph-nvmeof-gateway-5fc7679485-txntq)
[01-Aug-2025 14:18:47] DEBUG state.py:577 (1): OMAP was unlocked, thread id: 75, id: gateway-ceph-nvmeof-gateway-5fc7679485-txntq, cookie: omap_file_cookie_0x7f1b21241610_1_75_1754057927664901697_shared
[01-Aug-2025 14:18:47] DEBUG state.py:862 (1): Released OMAP file lock after reading content (gateway-ceph-nvmeof-gateway-5fc7679485-txntq)
[01-Aug-2025 14:18:47] DEBUG state.py:1413 (1): Check local version 2 against OMAP version 2 (gateway-ceph-nvmeof-gateway-5fc7679485-txntq).
[01-Aug-2025 14:18:49] DEBUG cephutils.py:82 (1): nvme-show string: {"prefix":"nvme-gw show", "pool":"ocs-storagecluster-cephblockpool", "group":"openshift-storage"}
[01-Aug-2025 14:18:49] DEBUG cephutils.py:39 (1): Execute monitor command: {"prefix":"nvme-gw show", "pool":"ocs-storagecluster-cephblockpool", "group":"openshift-storage"}
[01-Aug-2025 14:18:49] DEBUG cephutils.py:42 (1): Monitor reply: (0, b'{\n    "epoch": 1,\n    "pool": "ocs-storagecluster-cephblockpool",\n    "group": "openshift-storage",\n    "features": "LB",\n    "num gws": 0\n}\n', '')
[01-Aug-2025 14:18:49] DEBUG cephutils.py:84 (1): reply "(0, b'{\n    "epoch": 1,\n    "pool": "ocs-storagecluster-cephblockpool",\n    "group": "openshift-storage",\n    "features": "LB",\n    "num gws": 0\n}\n', '')"
[01-Aug-2025 14:18:49] INFO cephutils.py:92 (1): illegal rebalance ana group  0
[01-Aug-2025 14:18:49] DEBUG cephutils.py:95 (1): Rebalance ana_group: 0, num-gws: 0
[01-Aug-2025 14:18:49] WARNING cephutils.py:110 (1): GWs not found
```
