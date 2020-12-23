# Backup and Restore ETCD using docker CLI
1. The important folder:
   1. `/etc/kubernetes/pki/etcd` - certificates folder
      1. `ca.crt` -- ca certificates file
      2. `healthcheck-client.key` -- client certificates key file
      3. `healthcheck-client.crt` -- client certificates file
   2. `/home/hadn/backup` - folder on the host
2. Docker CLI
   1. Backup ETCD
      ```bash
      docker run --rm \
          -v $(pwd)/backup:/backup \
          --network host \
          -v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd \
          --env ETCDCTL_API=3  k8s.gcr.io/etcd:3.4.3-0 etcdctl \
          --endpoints=https://127.0.0.1:2379 \
          --cacert=/etc/kubernetes/pki/etcd/ca.crt \
          --cert=/etc/kubernetes/pki/etcd/server.crt \
          --key=/etc/kubernetes/pki/etcd/server.key \
          snapshot save /backup/etcd-snapshot-latest.db
      ```
   2. Check backup file
      ```bash
      docker run --rm \
          -v $(pwd)/backup:/backup \
          --network host \
          -v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd \
          --env ETCDCTL_API=3  k8s.gcr.io/etcd:3.4.3-0 etcdctl \
          --endpoints=https://127.0.0.1:2379 \
          --cacert=/etc/kubernetes/pki/etcd/ca.crt \
          --cert=/etc/kubernetes/pki/etcd/server.crt \
          --key=/etc/kubernetes/pki/etcd/server.key \
          --write-out=table snapshot status /backup/etcd-snapshot-latest.db
      ```
   3. Restore ETCD
      1. run command cli
      ```bash
      docker run --rm \
          -v $(pwd)/backup:/backup \
          -v $(pwd):/mnt \
          --env ETCDCTL_API=3 \
          k8s.gcr.io/etcd:3.4.3-0 \
          etcdctl snapshot restore /backup/etcd-snapshot-latest.db --data-dir /mnt/res-test

      # will restore to ./res folder
      # res
      # └── member
      #     ├── snap
      #     │   ├── 0000000000000001-0000000000000001.snap
      #     │   └── db
      #     └── wal
      #         └── 0000000000000000-0000000000000000.wal

      ```
	  2. edit etcd.yaml to use `restore location folder`
	 	 ```yaml
		 apiVersion: v1
		 kind: Pod
		 metadata:
		   annotations:
		 	 kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.2.150:2379
		   creationTimestamp: null
		   labels:
			 component: etcd
			 tier: control-plane
		   name: etcd
		   namespace: kube-system
		 spec:
		   containers:
		   - command:
			 - etcd
			 - --advertise-client-urls=https://192.168.2.150:2379
			 - --cert-file=/etc/kubernetes/pki/etcd/server.crt
			 - --client-cert-auth=true
			 - --data-dir=/var/lib/etcd
			 - --initial-advertise-peer-urls=https://192.168.2.150:2380
			 - --initial-cluster=master-1=https://192.168.2.150:2380
			 - --key-file=/etc/kubernetes/pki/etcd/server.key
			 - --listen-client-urls=https://127.0.0.1:2379,https://192.168.2.150:2379
			 - --listen-metrics-urls=http://127.0.0.1:2381
			 - --listen-peer-urls=https://192.168.2.150:2380
			 - --name=master-1
			 - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
			 - --peer-client-cert-auth=true
			 - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
			 - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
			 - --snapshot-count=10000
			 - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
			 image: k8s.gcr.io/etcd:3.4.13-0
			 imagePullPolicy: IfNotPresent
			 livenessProbe:
			   failureThreshold: 8
			   httpGet:
				 host: 127.0.0.1
				 path: /health
				 port: 2381
				 scheme: HTTP
			   initialDelaySeconds: 10
			   periodSeconds: 10
			   timeoutSeconds: 15
			 name: etcd
			 resources: {}
			 startupProbe:
			   failureThreshold: 24
			   httpGet:
				 host: 127.0.0.1
				 path: /health
				 port: 2381
				 scheme: HTTP
			   initialDelaySeconds: 10
			   periodSeconds: 10
			   timeoutSeconds: 15
			 volumeMounts:
			 - mountPath: /var/lib/etcd
			   name: etcd-data
			 - mountPath: /etc/kubernetes/pki/etcd
			   name: etcd-certs
		   hostNetwork: true
		   priorityClassName: system-node-critical
		   volumes:
		   - hostPath:
			   path: /etc/kubernetes/pki/etcd
			   type: DirectoryOrCreate
			 name: etcd-certs
		   - hostPath:
			   path: /root/res-test
			   # path: /var/lib/etcd
			   type: DirectoryOrCreate
			 name: etcd-data
		 status: {}
		 ```
      3. sudo -i
      4. systemctl stop kubelet
      5. systemctl stop docker
         1. Remove all stop docker containers `docker container stop $(docker container ls -aq)`
      6. systemctl start docker
      7. systemctl start kubelet
3. Reference link
   1. https://elastisys.com/backup-kubernetes-how-and-why/
