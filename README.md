# Percona XtraDB Cluster for Alpine (Experimental)

This is a temporary repo for testing PXC on Alpine.

This will only compile and work with Alpine v3.6 as libressl-dev in later versions are not compatible with galera.

Because of this PXC cannot be pushed to aport. Maybe when PXC starts using Mysql 8.0 bransh and supports libressl it will be possible.

If you have any issues just report it.

## Install instructions

```
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/5.7.22-29.26/percona-xtradb-cluster-common-5.7.22-r0.apk" -o "percona-xtradb-cluster-common-5.7.22-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/5.7.22-29.26/percona-xtradb-cluster-server-5.7.22-r0.apk" -o "percona-xtradb-cluster-server-5.7.22-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/5.7.22-29.26/percona-xtradb-cluster-client-5.7.22-r0.apk" -o "percona-xtradb-cluster-client-5.7.22-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/5.7.22-29.26/percona-xtradb-cluster-galera-5.7.22-r0.apk" -o "percona-xtradb-cluster-galera-5.7.22-r0.apk"
curl -fSL "https://github.com/Flowman/pxc-alpine/releases/download/5.7.16-27.19/percona-xtrabackup-2.4.6-r0.apk" -o "percona-xtrabackup-2.4.6-r0.apk"

apk add --allow-untrusted \
    percona-xtradb-cluster-common-5.7.22-r0.apk \
    percona-xtradb-cluster-client-5.7.22-r0.apk \
    percona-xtradb-cluster-galera-5.7.22-r0.apk \
    percona-xtrabackup-2.4.6-r0.apk \
    percona-xtradb-cluster-server-5.7.22-r0.apk
```