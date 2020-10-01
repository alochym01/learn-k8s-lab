# Lab backup and restore Single ETCD
## Backup Certificates
1.  cp -r /etc/kubernetes/pki /opt/alochym/certs-backup
## Backup ETCD
1.  Create a snapshot etcd server using etcdctl cli
    1.  go get github.com/coreos/etcd/etcdctl - should be same etcd version of etcd.yaml in /etc/kubernetes/manifests/etcd.yaml
    2.  save a snapshot
        ```bash
        ETCDCTL_API=3 \
        etcdctl snapshot save /opt/alochym/etcd-snapshot.db \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/healcheck-client.crt
        --key=/etc/kubernetes/pki/etcd/healcheck-client.key
        ```
2.  Testing
    1.  reset everything of kubernetes - `kubeadm reset -f`
    2.  copy everything backup certs to /etc/kubernetes/ - `cp -r /opt/alochym/certs-backup /etc/kubernetes/`
    3.  restore etcd server
        ```bash
        ETCDCTL_API=3 \
        etcdctl snapshot restore /opt/alochym/etcd-snapshot.db

        all restore to $PWD/default.etcd folder
        ```
    4.  Move all default.etcd/member to /var/lib/etcd - `mv default.etcd/member /var/lib/etcd`
    5.  Initialize kubernetes
        ```bash
        kubeadm init --ignore-preflight-errors=DirAvailable--var-lib-etcd
        ```
