# Contributor: Peter Szalatnay <theotherland@gmail.com>
# Maintainer: Peter Szalatnay <theotherland@gmail.com>
pkgname=percona-xtradb-cluster
_pkgname=Percona-XtraDB-Cluster
pkgver=5.7.22
_pkgver="$pkgver-29.26"
pkgrel=0
pkgdesc="Percona XtraDB Cluster is an active/active high availability and high scalability open source solution for MySQL clustering"
url="https://www.percona.com/software/mysql-database/percona-xtradb-cluster"
pkgusers="mysql"
pkggroups="mysql"
arch="all"
license="GPL2"
depends="$pkgname-common $pkgname-server $pkgname-client"
depends_dev="openssl-dev zlib-dev"
makedepends="cmake openssl-dev zlib-dev readline-dev libaio-dev ncurses-dev linux-headers bison bsd-compat-headers rpcgen portablexdr-dev"
install="$pkgname.pre-install"
source="https://github.com/percona/percona-xtradb-cluster/archive/$_pkgname-$_pkgver.tar.gz
        fix-posix_timers.patch
        my.cnf
        my.cnf.old
        percona-xtradb-cluster.cnf
        mysqld.cnf
        mysqld_safe.cnf
        client.cnf
        wsrep.cnf
        "

subpackages="$pkgname-doc $pkgname-dev $pkgname-common $pkgname-client-libs:_client_libs
        $pkgname-client $pkgname-server $pkgname-test:mytest"

builddir="$srcdir/$pkgname-$_pkgname-$_pkgver"

# Notes:
# PXC require boost 1.59 or it will now compile, alpine edge uses boost 1.66 hence using the download flag
build() {
        cd "$builddir"

        source "VERSION"
        local MYSQL_VERSION="$MYSQL_VERSION_MAJOR.$MYSQL_VERSION_MINOR.$MYSQL_VERSION_PATCH"
        local PERCONA_SERVER_EXTENSION="$(echo $MYSQL_VERSION_EXTRA | sed 's/^-/rel/')"

        local WSREP_VERSION="$(grep WSREP_INTERFACE_VERSION wsrep/wsrep_api.h | cut -d '"' -f2).$(grep 'SET(WSREP_PATCH_VERSION' "cmake/wsrep.cmake" | cut -d '"' -f2)"

        local COMMENT="Percona XtraDB Cluster binary (GPL) $MYSQL_VERSION-$WSREP_VERSION"

        CFLAGS="$CFLAGS -DPERCONA_INNODB_VERSION=$PERCONA_SERVER_EXTENSION -DSIGEV_THREAD_ID=4 -U_FORTIFY_SOURCE"
        CXXFLAGS="$CXXFLAGS -DPERCONA_INNODB_VERSION=$PERCONA_SERVER_EXTENSION -U_FORTIFY_SOURCE"

        cmake . -DBUILD_CONFIG=mysql_release \
                -DCMAKE_BUILD_TYPE=RelWithDebInfo \
                -DSYSCONFDIR=/etc/mysql \
                -DWITH_EMBEDDED_SERVER=OFF \
                -DFEATURE_SET=community \
                -DENABLE_DTRACE=OFF \
                -DWITH_SSL=system \
                -DWITH_ZLIB=system \
                -DCMAKE_INSTALL_PREFIX=/usr \
                -DMYSQL_DATADIR=/var/lib/mysql \
                -DINSTALL_INFODIR=share/mysql/docs \
                -DINSTALL_MANDIR=share/man \
                -DINSTALL_PLUGINDIR=lib/mysql/plugin \
                -DINSTALL_SCRIPTDIR=bin \
                -DINSTALL_INCLUDEDIR=include/mysql \
                -DINSTALL_DOCREADMEDIR=share/mysql \
                -DINSTALL_SUPPORTFILESDIR=share/mysql \
                -DINSTALL_MYSQLSHAREDIR=share/mysql \
                -DINSTALL_DOCDIR=share/mysql/docs \
                -DINSTALL_SHAREDIR=share/mysql \
                -DMYSQL_UNIX_ADDR=/run/mysqld/mysqld.sock \
                -DDEFAULT_CHARSET=utf8 \
                -DDEFAULT_COLLATION=utf8_general_ci \
                -DMYSQL_SERVER_SUFFIX="-$WSREP_VERSION" \
                -DWITH_INNODB_DISALLOW_WRITES=ON \
                -DDOWNLOAD_BOOST=1 \
                -DWITH_BOOST="$builddir"/libboost \
                -DWITH_WSREP=ON \
                -DWITH_READLINE=system \
                -DWITHOUT_TOKUDB=ON \
                -DWITHOUT_ROCKSDB=ON \
                -DWITH_KEYRING_VAULT=0 \
                -DCOMPILATION_COMMENT="$COMMENT" \
                -DWITH_PAM=OFF \
                -DWITH_INNODB_MEMCACHED=ON \
                -DWITH_SCALABILITY_METRICS=ON \

        make
}

check() {
    cd "$builddir"
    make test
}

package() {
        cd "$builddir"
        make DESTDIR="$pkgdir/" install

        install -Dm644 COPYING "$pkgdir"/usr/share/licenses/$pkgname/COPYING

        # PXC ubuntu package include the mysql-common conf, but we do not want to use it as our default conf
        install -Dm640 -o mysql "$srcdir"/my.cnf \
            "$pkgdir"/etc/mysql/my.cnf
        install -Dm640 -o mysql "$srcdir"/my.cnf.old \
            "$pkgdir"/etc/mysql/my.cnf.old
        install -Dm640 -o mysql "$srcdir"/percona-xtradb-cluster.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.cnf

        mkdir -p "$pkgdir"/etc/mysql/conf.d/

        install -Dm640 -o mysql "$srcdir"/client.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/client.cnf
        install -Dm640 -o mysql "$srcdir"/mysqld.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/mysqld.cnf
        install -Dm640 -o mysql "$srcdir"/mysqld_safe.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/mysqld_safe.cnf
        install -Dm640 -o mysql "$srcdir"/wsrep.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/wsrep.cnf

        # remove xinetd.d as its not used in alpine
        rm -rf "$pkgdir"/usr/xinetd.d

        # mysql-test includes one executable that doesn't belong under
        # /usr/share, so move it and provide a symlink
        mv "$pkgdir"/usr/mysql-test/lib/My/SafeProcess/my_safe_process \
            "$pkgdir"/usr/bin
        ln -s ../../../../bin/my_safe_process \
            "$pkgdir"/usr/mysql-test/lib/My/SafeProcess/my_safe_process
}

doc() {
    mkdir -p "$subpkgdir"/usr/share
    mv "$pkgdir"/usr/share/man "$subpkgdir"/usr/share
    default_doc
}

common() {
    pkgdesc="Percona XtraDB Cluster common files for both server and client"
    replaces="mysql-common mariadb-common"
    depends=
    install -Dm750 -o mysql -d "$subpkgdir"/var/log/mysql
    install -Dm750 -o mysql -d "$subpkgdir"/usr/lib/mysql
    install -Dm750 -o mysql -d "$subpkgdir"/run/mysqld
    mkdir -p "$subpkgdir"/usr/share/mysql \
        "$subpkgdir"/etc \
        "$subpkgdir"/usr/lib/mysql/plugin
    mv "$pkgdir"/etc/mysql "$subpkgdir"/etc/
    local lang="charsets danish english french greek italian korean norwegian-ny
        portuguese russian slovak swedish czech dutch estonian german
        hungarian japanese norwegian polish romanian serbian spanish
        ukrainian bulgarian"
    for l in $lang; do
        mv "$pkgdir"/usr/share/mysql/$l \
            "$subpkgdir"/usr/share/mysql/
    done
}

_client_libs() {
    pkgdesc="Percona XtraDB Cluster client library"
    replaces="percona-xtradb-cluster libmysqlclient"
    depends="$pkgname-common"
    mkdir -p "$subpkgdir"/usr/lib \
        "$subpkgdir"/usr/share/mysql
    mv "$pkgdir"/usr/lib/libperconaserverclient.so* \
        "$subpkgdir"/usr/lib/
}

mytest() {
    pkgdesc="The test suite distributed with Percona XtraDB Cluster"
    mkdir -p "$subpkgdir"/usr/bin
    mv "$pkgdir"/usr/bin/mysql_client_test \
        "$pkgdir"/usr/mysql-test \
        "$pkgdir"/usr/bin/my_safe_process \
        "$subpkgdir"/usr/bin/
}

client() {
    pkgdesc="client for the Percona XtraDB Cluster database"
    depends="$pkgname-common perl-dbi"
    replaces="mysql-client mariadb-client"
    mkdir -p "$subpkgdir"/usr/bin/
    local bins="myisam_ftdump mysql mysqladmin
        mysqlcheck mysqldump mysqldumpslow
        mysqlimport mysqlshow mysqlslap mysql_config_editor"
    for i in $bins; do
        mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/
    done
}

server() {
    pkgdesc="server for the Percona XtraDB Cluster database"
    depends="$pkgname-common $pkgname-galera percona-xtrabackup socat iproute2
        procps findutils coreutils tzdata bash perl-dbd-mysql"
    replaces="mariadb"
    mkdir -p "$subpkgdir"/usr/lib/mysql/plugin
    mv "$pkgdir"/usr/lib/mysql/plugin/*.so \
        "$subpkgdir"/usr/lib/mysql/plugin/
    local bins="mysqld clustercheck pyclustercheck innochecksum my_print_defaults
        myisamchk myisamlog myisampack mysql_install_db mysql_secure_installation
        mysql_tzinfo_to_sql mysql_upgrade mysql_plugin mysqlbinlog mysqld_multi
        mysqld_safe mysqltest perror replace resolve_stack_dump resolveip
        wsrep_sst_common wsrep_sst_mysqldump wsrep_sst_xtrabackup-v2 wsrep_sst_rsync
        mysql_ssl_rsa_setup"
    mkdir -p "$subpkgdir"/usr/bin/
    for i in $bins; do
        mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/
    done
}

