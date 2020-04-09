# Volume

The Kubernetes Volume abstraction solve two problems

- when a container crash, kubelet will restart it, but the files will be lost

- containers in a Pod need to share files

## Background

1. docker volumes is somewhat looser and less managed
   
   1. docker volume is simply a directory on disk or in another container
   
   2. lifetime are not managed
   
   3. docker now provide volume drivers, but the functionality is very limited,

2. kubernetes volumes
   
   1. explicit lifetime: the same as the Pod than enclose it
   
   2. support many types of volumes,  a Pod can use any number of them simultaneously
   
   3. to use a volume, a Pod specifies what volumes to provide for the Pod (.spec.volumes field) and where to mount this into Containers (the .spec.containers[*].volumeMounts field)
   
   4. Volumes can not mount onto other Volumes or have hard links to other Volumes. 

## Types of Volumes

- awsElasticBlockStore

- azureDisk

- azureFile

- cephfs

- cinder

- configMap

- emptyDir

- csi

- hostPath

- iscsi

- local

- nfs

- persistentVolumeClaim

- projected

- rbd

- secret

### configMap

the configMap resource provide a way to inject configuration data into Pods. the data stored in a ConfigMap object can be referenced in a voume of type configMap.

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```

the log-config ConfigMap is mounted as a volume, and all contents stored in its log_level entry are mounted into Pod at path: "/etc/config/log_level"

*Caution: You must create a ConfigMap before you can use it*

*Note: A container using a ConfigMap as a subPath volume will not receive ConfigMap updates*

### emptyDir

An emptyDir volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is runnint on that node. When the Pod is removed from a node for any reason , the data in emptyDir is deleted forever.

By default, `emptyDir` volumes are stored on whatever medium is backing the node,  however, you can set the `emptyDir.medium` field to `"Memory"` to tell Kubernetes to mount a tmpfs (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that unlike disks, tmpfs is cleared on node reboot and any files you write will count against your Container’s memory limit

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

*note: no need to specify the emptyDir path on host*

*note: different with hostPath*

### hostPath

A `hostPath` volume mounts a file or directory from the host node’s filesystem into your Pod

In addition to the required `path` property, user can optionally specify a `type` for a hostPath volume.  The supported values for filed `type` are:

| value             | behaviro                                                                                                         |
| ----------------- |:---------------------------------------------------------------------------------------------------------------- |
|                   | Empty string(default), no checks will be performed before mounting                                               |
| DirectoryOrCreate | If nothing exists , an empty directory(0755) will be created , having the same group and ownership with Kubelet. |
| Directory         | directory must exist                                                                                             |
| FileOrCreate      | If nothing exists, an empty file(0644) will be created, having the same group and ownership with Kubelet.        |
| File              | file must exist                                                                                                  |
| Socket            | UNIX   socket must exist                                                                                         |
| CharDevice        | character device must exist                                                                                      |
| BlockDevice       | block device must exist                                                                                          |

*Caution: the `FileOrCreate` mode does not create the parent directory of the file*

*todo: DirectoryOrCreate will create the parent directory???, need to check*

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-webserver
    image: k8s.gcr.io/test-webserver:latest
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # Ensure the file directory is created.
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```

### local

### persistentVolumeClaim

A `persistentVolumeClaim` volume is to mount a `PersistentVolume` into a Pod. PersistentVolumes are a way for user to claims durable storage. 

### projected

A `projected` volume maps serveral existing volume sources into the same directory.

types of volume sources can be projected:

- secret

- downwardAPI

- configMap

- serviceAccountToken

All sources are required to be in the same namespace as the Pod. 

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "cpu_limit"
              resourceFieldRef:
                containerName: container-test
                resource: limits.cpu
      - configMap:
          name: myconfigmap
          items:
            - key: config
              path: my-group/my-config
```

### rbd

An `rbd` volume allows a Rados Block Device volume to  be mounted into the Pod. 

### secret

A `secret` volume is used to pass sensitive information, such as passwords, to Pods. You can store secrets in the Kubernetes API and mount them as files for use by Pods without coupling to Kubernetes directly. `secret` volumes are backed by tmpfs (a RAM-backed filesystem) so they are never written to non-volatile storage

*Caution: User must create a Secret in the Kubernetes API before using it*

*Note: A container using a subPath volume mount will not receive Secret updates*

### downwardAPI

A `downwardAPI` volume is used to make downward API data available to applications. It mounts a directory and writes the requested data in plain text files

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
```

## Using subPath

Sometime, it is useful to share one volume for multiple uses in a single Pod. The volumeMount.subPath property can be used to specify a sub-path inside the  referenced volume instead of its root.

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```

Using subPath with expanded environment variables(v1.17 stable)

Use the `subPathExpr` field to construct `subPath` directory names from Downward API environment variables. This feature requires the `VolumeSubpathEnvExpansion` feature gate to be enabled. It is enabled by default starting with Kubernetes 1.15. The `subPath` and `subPathExpr` properties are mutually exclusive.

example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```
