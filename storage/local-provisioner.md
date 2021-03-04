# Config server
1.  Create a disk with mount point `/mnt/disk`
## Storage Class
1.  Create Storage class with `provisioner: kubernetes.io/no-provisioner`
    ```yaml
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
        name: fast-disks
    provisioner: kubernetes.io/no-provisioner
    volumeBindingMode: WaitForFirstConsumer
    ```
## Config Persistent Volume
1.  We create a 3 persistent volumes which are used in statefulset with 3 replicate
2.  pv-0.yaml
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: my-local-pv-0
    spec:
        capacity:
            storage: 1Gi
        accessModes:
        -   ReadWriteOnce
        persistentVolumeReclaimPolicy: Retain
        storageClassName: fast-disks
        local:
            path: /mnt/disk/redis-0
        # nodeAffinity:
        #     required:
        #         nodeSelectorTerms:
        #         -   matchExpressions:
        #             -   key: kubernetes.io/hostname
        #                 operator: In
        #                 values:
        #                 -   master-2
    ```
3.  pv-1.yaml
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: my-local-pv-1
    spec:
        capacity:
            storage: 1Gi
        accessModes:
        -   ReadWriteOnce
        persistentVolumeReclaimPolicy: Retain
        storageClassName: fast-disks
        local:
            path: /mnt/disk/redis-1
        # nodeAffinity:
        #     required:
        #         nodeSelectorTerms:
        #         -   matchExpressions:
        #             -   key: kubernetes.io/hostname
        #                 operator: In
        #                 values:
        #                 -   master-2
    ```
4.  pv-2.yaml
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: my-local-pv-2
    spec:
        capacity:
            storage: 1Gi
        accessModes:
        -   ReadWriteOnce
        persistentVolumeReclaimPolicy: Retain
        storageClassName: fast-disks
        local:
            path: /mnt/disk/redis-2
        # nodeAffinity:
        #     required:
        #         nodeSelectorTerms:
        #         -   matchExpressions:
        #             -   key: kubernetes.io/hostname
        #                 operator: In
        #                 values:
        #                 -   master-2
    ```
## Create StatefulSet
1.  redis config map
    1.  https://raw.githubusercontent.com/marcel-dempers/docker-development-youtube-series/master/storage/redis/kubernetes/redis/redis-configmap.yaml
2.  redis statefulset
    1.  https://raw.githubusercontent.com/marcel-dempers/docker-development-youtube-series/master/storage/redis/kubernetes/redis/redis-statefulset.yaml
3.  kubectl apply -f redis-configmap.yaml
3.  kubectl apply -f redis-statefulset.yaml
