# Deployment for the ceph-nvme gateway on OpenShift Data Foundation

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
