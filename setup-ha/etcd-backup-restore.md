# Lab backup and restore Single ETCD
## Backup Certificates
1.  cp -r /etc/kubernetes/pki /opt/alochym/certs-backup
## Backup ETCD
1.  Create a snapshot etcd server using etcdctl cli
    1.  copy etcdctl from pod in `kube-system` namespace - `kubectl cp kube-system/etcd-master-1:/usr/local/bin/etcdctl etcd/etcdctl`
    2.  save a snapshot. if you have a cluster, should restore for 3 times and then start cluster
        ```bash
        ETCDCTL_API=3 \
        ./etcdctl snapshot save /opt/alochym/etcd-snapshot.db \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt
        --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
        ```
    3.  Test backup file
        ```bash
        root@master-1:/home/hadn/etcd# ./etcdctl --write-out=table snapshot status etcd-snapshot.db
        +----------+----------+------------+------------+
        |   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
        +----------+----------+------------+------------+
        | c180d0f6 | 17336887 |       1639 |     5.0 MB |
        +----------+----------+------------+------------+
        ```
2.  Testing
    1.  reset everything of kubernetes - `kubeadm reset -f`
    2.  copy everything backup certs to /etc/kubernetes/ - `cp -r /opt/alochym/certs-backup /etc/kubernetes/`
    3.  restore etcd server
        ```bash
        ETCDCTL_API=3 \
        ./etcdctl snapshot restore /opt/alochym/etcd-snapshot.db

        all restore to $PWD/default.etcd folder
        ```
    4.  Move all default.etcd/member to /var/lib/etcd - `mv default.etcd/member /var/lib/etcd`
    5.  Initialize kubernetes
        ```bash
        kubeadm init --ignore-preflight-errors=DirAvailable--var-lib-etcd
        ```
