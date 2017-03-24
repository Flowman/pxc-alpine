# Percona XtraDB Cluster for Alpine (Experimental)

This is a temporary repo for testing PXC on Alpine. It will be submitted to the Alpine repo when its done.

If you have any issues just report it.

## Install instructions

```
wget https://github.com/Flowman/pxc-alpine/releases/download/5.7.16-27.19/percona-xtradb-cluster-common-5.7.16-r0.apk
wget https://github.com/Flowman/pxc-alpine/releases/download/5.7.16-27.19/percona-xtradb-cluster-galera-5.7.16-r0.apk
wget https://github.com/Flowman/pxc-alpine/releases/download/5.7.16-27.19/percona-xtrabackup-2.4.6-r0.apk
wget https://github.com/Flowman/pxc-alpine/releases/download/5.7.16-27.19/percona-xtradb-cluster-server-5.7.16-r0.apk

apk add --allow-untrusted \
	percona-xtradb-cluster-common-5.7.16-r0.apk \
	percona-xtradb-cluster-galera-5.7.16-r0.apk \
	percona-xtrabackup-2.4.6-r0.apk \
    percona-xtradb-cluster-server-5.7.16-r0.apk
```