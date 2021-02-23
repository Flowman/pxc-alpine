# Percona XtraDB Cluster for Alpine (Experimental)

This is a temporary repo for testing PXC on Alpine. Started work on the 8.0 branch

If you have any issues just report it.

## Install instructions

```
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/8.0.21-12.1/percona-xtradb-cluster-common-8.0.21-r0.apk" -o "percona-xtradb-cluster-common-8.0.21-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/8.0.21-12.1/percona-xtradb-cluster-server-8.0.21-r0.apk" -o "percona-xtradb-cluster-server-8.0.21-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/8.0.21-12.1/percona-xtradb-cluster-client-8.0.21-r0.apk" -o "percona-xtradb-cluster-client-8.0.21-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/8.0.22-15/percona-xtrabackup-8.0.22-r0.apk" -o "percona-xtrabackup-8.0.22-r0.apk"

apk add --allow-untrusted \
    percona-xtradb-cluster-common-8.0.21-r0.apk \
    percona-xtradb-cluster-client-8.0.21-r0.apk \
    percona-xtrabackup-8.0.22-r0.apk \
    percona-xtradb-cluster-server-8.0.21-r0.apk
```
