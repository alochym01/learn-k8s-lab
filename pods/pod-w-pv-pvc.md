# Lab Pod with Persistent Volume and Persistent Volume Claim
## Requirement
0.  Create folder for hostPath to mount on worker-1 - `/home/hadn/volume`
1.  Create Persistent Volume with hostPath
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
        name: pv-volume
        labels:
            type: local
    spec:
        # Retain, Delete, Recycle
        # Recycle policy is deprecated. Instead, the recommended approach is to use dynamic provisioning.
        persistentVolumeReclaimPolicy: Recycle # what happens PersistentVolumeClaim is deleted
        capacity:
            storage: 10Gi    # The configuration also specifies a size of 10 gibibytes
        accessModes:
            - ReadWriteOnce    # access mode of ReadWriteOnce
        hostPath:
            path: "/home/hadn/volume"  # folder /home/hadn/volume must exist on the Node
    ```
    1.  Check result - `kubectl get pv -owide`
        ```bash
        NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE   VOLUMEMODE
        task-pv-volume   10Gi       RWO            Recycle          Available                                   2s    Filesystem
        ```
2.  Create Persistent Volume Claim, which uses Persistent Volume in step 1
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: pvc-volume
    spec:
        accessModes:
            - ReadWriteOnce
        resources:
            requests:
                storage: 3Gi
    ```
    1.  Check result - `kubectl get pv,pvc`
        ```yaml
        NAME                              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
        persistentvolume/task-pv-volume   10Gi       RWO            Recycle          Bound    default/task-pv-claim                           2m51s

        NAME                                  STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
        persistentvolumeclaim/task-pv-claim   Bound    task-pv-volume   10Gi       RWO                           23s
        ```
3.  Create Pod with using Persistent Volume Claim in step 2
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
        name: pv-pod
    spec:
        volumes:
            -   name: pv-storage
                persistentVolumeClaim:
                    claimName: pvc-volume # Mounting PersistentVolumeClaim to POD
        containers:
            - name: pv-container
                image: nginx
                ports:
                    -   containerPort: 80
                        name: "http-server"
                volumeMounts:
                    -   mountPath: "/usr/share/nginx/html"
                        name: pv-storage  # Mounting PODS Volume to Container
    ```
    1.  Create some files
        -   `kubectl exec -it pv-pod -- sh`
        -   `env | tee /usr/share/nginx/html/alochym.txt`
4.  Check directory on Node which is hosted hostPath - `cat /home/hadn/volume/alochym.txt`
    ```bash
    KUBERNETES_SERVICE_PORT=443
    KUBERNETES_PORT=tcp://10.96.0.1:443
    HOSTNAME=pv-pod
    HOME=/root
    PKG_RELEASE=1~buster
    TERM=xterm
    KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    KUBERNETES_PORT_443_TCP_PORT=443
    NJS_VERSION=0.4.3
    KUBERNETES_PORT_443_TCP_PROTO=tcp
    KUBERNETES_SERVICE_PORT_HTTPS=443
    KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
    KUBERNETES_SERVICE_HOST=10.96.0.1
    PWD=/
    ```
