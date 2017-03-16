# Contributor: Peter Szalatnay <theotherland@gmail.com>
# Maintainer: Peter Szalatnay <theotherland@gmail.com>
pkgname=percona-xtradb-cluster
_pkgname=Percona-XtraDB-Cluster
pkgver=5.7.16
_pkgver=5.7.16-27.19
pkgrel=0
pkgdesc="Percona XtraDB Cluster"
url="http://www.percona.com"
pkgusers="mysql"
pkggroups="mysql"
arch="all"
license="GPL2"
depends="$pkgname-common"
depends_dev="openssl-dev zlib-dev"
makedepends="cmake openssl-dev zlib-dev readline-dev libaio-dev ncurses-dev linux-headers bison bsd-compat-headers"
install="$pkgname.pre-install"
source="https://github.com/percona/percona-xtradb-cluster/archive/$_pkgname-$_pkgver.tar.gz
        fix-posix_timers.patch
        percona-xtradb-cluster.cnf
        mysqld.cnf
        mysqld_safe.cnf
        client.cnf
        wsrep.cnf
        "

subpackages="$pkgname-doc $pkgname-dev $pkgname-common $pkgname-test:mytest
        "

_builddir="$srcdir/$pkgname-$_pkgname-$_pkgver"
prepare() {
        local i
        cd "$_builddir"
        for i in $source; do
                case $i in
                *.patch) msg $i; patch -p1 -i "$srcdir"/$i || return 1;;
                esac
        done
}

build() {
        cd "$_builddir"

        source "VERSION"
        local MYSQL_VERSION="$MYSQL_VERSION_MAJOR.$MYSQL_VERSION_MINOR.$MYSQL_VERSION_PATCH"
        local PERCONA_SERVER_EXTENSION="$(echo $MYSQL_VERSION_EXTRA | sed 's/^-/rel/')"

        local WSREP_VERSION="$(grep WSREP_INTERFACE_VERSION wsrep/wsrep_api.h | cut -d '"' -f2).$(grep 'SET(WSREP_PATCH_VERSION' "cmake/wsrep.cmake" | cut -d '"' -f2)"

        local COMMENT="Percona XtraDB Cluster binary (GPL) $MYSQL_VERSION-$WSREP_VERSION"

        local CFLAGS="-DPERCONA_INNODB_VERSION=$PERCONA_SERVER_EXTENSION -DSIGEV_THREAD_ID=4 -U_FORTIFY_SOURCE"
        local CXXFLAGS="-DPERCONA_INNODB_VERSION=$PERCONA_SERVER_EXTENSION -U_FORTIFY_SOURCE"

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
                -DWITH_BOOST="$_builddir"/libboost \
                -DWITH_WSREP=ON \
                -DWITH_UNIT_TESTS=0 \
                -DWITH_READLINE=system \
                -DWITHOUT_TOKUDB=ON \
                -DCOMPILATION_COMMENT="$COMMENT" \
                -DWITH_PAM=OFF \
                -DWITH_INNODB_MEMCACHED=ON \
                -DWITH_SCALABILITY_METRICS=ON \
                || return 1

        make || return 1
}

package() {
        cd "$_builddir"
        make DESTDIR="$pkgdir/" install || return 1

        install -Dm644 COPYING "$pkgdir"/usr/share/licenses/$pkgname/COPYING || return 1

        install -Dm640 -o mysql "$srcdir"/percona-xtradb-cluster.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.cnf

        ln -s /etc/mysql/percona-xtradb-cluster.cnf \
            "$pkgdir"/etc/mysql/my.cnf || return 1

        mkdir -p "$pkgdir"/etc/mysql/conf.d/ || return 1

        install -Dm640 -o mysql "$srcdir"/client.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/client.cnf || return 1
        install -Dm640 -o mysql "$srcdir"/mysqld.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/mysqld.cnf || return 1
        install -Dm640 -o mysql "$srcdir"/mysqld_safe.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/mysqld_safe.cnf || return 1
        install -Dm640 -o mysql "$srcdir"/wsrep.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.conf.d/wsrep.cnf || return 1

        install -Dm750 -o mysql -d "$pkgdir"/var/log/mysql || return 1
        install -Dm750 -o mysql -d "$pkgdir"/usr/lib/mysql || return 1
        install -Dm750 -o mysql -d "$pkgdir"/run/mysqld || return 1

        # mysql-test includes one executable that doesn't belong under
        # /usr/share, so move it and provide a symlink
        mv "$pkgdir"/usr/mysql-test/lib/My/SafeProcess/my_safe_process \
            "$pkgdir"/usr/bin
        ln -s ../../../../bin/my_safe_process \
            "$pkgdir"/usr/mysql-test/lib/My/SafeProcess/my_safe_process
}

doc() {
    mkdir -p "$subpkgdir"/usr/share
    mv "$pkgdir"/usr/man "$subpkgdir"/usr/share
    default_doc
}

common() {
    pkgdesc="Percona-XtraDB-Cluster common files for boh server and client"
    replaces="mysql-common"
    depends=
    mkdir -p "$subpkgdir"/usr/share/mysql "$subpkgdir"/etc
    mv "$pkgdir"/etc/mysql "$subpkgdir"/etc/ || return 1
    local lang="charsets danish english french greek italian korean norwegian-ny
        portuguese russian slovak swedish czech dutch estonian german
        hungarian japanese norwegian polish romanian serbian spanish
        ukrainian bulgarian"
    for l in $lang; do
        mv "$pkgdir"/usr/share/mysql/$l \
            "$subpkgdir"/usr/share/mysql/ || return 1
    done
}

mytest() {
    pkgdesc="The test suite distributed with Percona-XtraDB-Cluster"
    mkdir -p "$subpkgdir"/usr/bin || return 1
    mv "$pkgdir"/usr/bin/mysql_client_test \
        "$pkgdir"/usr/mysql-test \
        "$pkgdir"/usr/bin/my_safe_process \
        "$subpkgdir"/usr/bin/ \
        || return 1
}