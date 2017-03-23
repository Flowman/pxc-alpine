# Contributor: Peter Szalatnay <theotherland@gmail.com>
# Maintainer: Peter Szalatnay <theotherland@gmail.com>
pkgname=percona-xtradb-cluster
_pkgname=Percona-XtraDB-Cluster
pkgver=5.7.16
_pkgver=5.7.16-27.19
pkgrel=0
pkgdesc="Percona XtraDB Cluster is an active/active high availability and high scalability open source solution for MySQL clustering"
url="https://www.percona.com/software/mysql-database/percona-xtradb-cluster"
pkgusers="mysql"
pkggroups="mysql"
arch="all"
license="GPL2"
depends="$pkgname-common $pkgname-server $pkgname-galera percona-xtrabackup procps findutils coreutils socat iproute2 tzdata bash"
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

subpackages="$pkgname-doc $pkgname-dev $pkgname-common $pkgname-client $pkgname-server $pkgname-test:mytest
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

        install -Dm640 -o mysql "$srcdir"/my.cnf \
            "$pkgdir"/etc/mysql/my.cnf || return 1
        install -Dm640 -o mysql "$srcdir"/percona-xtradb-cluster.cnf \
            "$pkgdir"/etc/mysql/percona-xtradb-cluster.cnf

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
    mv "$pkgdir"/usr/share/man "$subpkgdir"/usr/share
    default_doc
}

common() {
    pkgdesc="Percona XtraDB Cluster common files for both server and client"
    replaces="mysql-common mariadb-common"
    depends=
    mkdir -p "$subpkgdir"/usr/share/mysql \
        "$subpkgdir"/etc \
        "$subpkgdir"/usr/lib/mysql/plugin
    mv "$pkgdir"/etc/mysql "$subpkgdir"/etc/ || return 1
    mv "$pkgdir"/usr/lib/mysql/plugin/dialog.so \
        "$pkgdir"/usr/lib/mysql/plugin/mysql_clear_password.so \
        "$subpkgdir"/usr/lib/mysql/plugin/ || return 1
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
    pkgdesc="The test suite distributed with Percona XtraDB Cluster"
    mkdir -p "$subpkgdir"/usr/bin || return 1
    mv "$pkgdir"/usr/bin/mysql_client_test \
        "$pkgdir"/usr/mysql-test \
        "$pkgdir"/usr/bin/my_safe_process \
        "$subpkgdir"/usr/bin/ \
        || return 1
}

client() {
    pkgdesc="client for the Percona XtraDB Cluster database"
    depends="$pkgname-common"
    replaces="mysql-client mariadb-client"
    local bins="myisam_ftdump mysql mysqladmin
        mysqlcheck mysqldump mysqldumpslow
        mysqlimport mysqlshow mysqlslap mysql_config_editor"
    mkdir -p "$subpkgdir"/usr/bin/ || return 1
    for i in $bins; do
        mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/ || return 1
    done
}

server() {
    pkgdesc="server for the Percona XtraDB Cluster database"
    depends="$pkgname-common"
    replaces="mariadb"
    mkdir -p "$subpkgdir"/usr/lib/mysql/plugin
    mv "$pkgdir"usr/lib/mysql/plugin/*.so
        "$subpkgdir"/usr/lib/mysql/plugin/
    local bins="mysqld clustercheck pyclustercheck innochecksum my_print_defaults
        myisamchk myisamlog myisampack mysql_install_db mysql_secure_installation
        mysql_tzinfo_to_sql mysql_upgrade mysql_plugin mysqlbinlog mysqld_multi
        mysqld_safe mysqltest perror replace resolve_stack_dump resolveip
        wsrep_sst_common wsrep_sst_mysqldump wsrep_sst_xtrabackup-v2 wsrep_sst_rsync"
    mkdir -p "$subpkgdir"/usr/bin/ || return 1
    for i in $bins; do
        mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/ || return 1
    done
}

md5sums="58f8524be1e25cc740cfa009d3e7c09c  Percona-XtraDB-Cluster-5.7.16-27.19.tar.gz
bae0925c27db3c154b7749fe5737bd1d  fix-posix_timers.patch
8c1516e62b6d6d1468f2c218d429ef28  fix-wsrep_sst_xtrabackup-v2.patch
4cad1109e85f2d99cb67813e61ad7f26  percona-xtradb-cluster.cnf
824cabcda6a881161faf6ae9839e5032  mysqld.cnf
82a9c756367888bbc5de4eb7481c778d  mysqld_safe.cnf
3f6cd33da04024ac92929fb696f4b1d8  client.cnf
8a5d6d69c7e6467cdbf9810d7ff91982  wsrep.cnf"
sha256sums="2c72168d4d6c2b0f029a8a88b1b6e3d1520133cac788e045ff641ab4e0db8135  Percona-XtraDB-Cluster-5.7.16-27.19.tar.gz
2c3fdccadf30fb3f1258663efaf5c60526236b56463f8d29c442107ac537488f  fix-posix_timers.patch
b2cdbcc45280ae0c13578a50e46e8aa8163551ab06957c6887e88a721da2e49c  fix-wsrep_sst_xtrabackup-v2.patch
8eee310ddaf5ffa20ab2f060ec7ecc8d0ff7b1d101f6f76c65cbcfb13a8bcd91  percona-xtradb-cluster.cnf
e06effa8275bef218eba830f7480733ff47509e4ef14b8d9e7e74aad2890dc15  mysqld.cnf
3208ead8094591813c7fe6a564e3826fd544394726ef0542de56b522332213df  mysqld_safe.cnf
6fadf39a263fa779293aeb38b23a2bd807a0e2a297659db082af1166a2b5af2b  client.cnf
9adf6f327c0dbd870842a9bb5d99ca271420384ff8b22b3e30e8f4d95753e616  wsrep.cnf"
sha512sums="2ec1ab64321e30a0564da9b5ea17fc125d51cf4d1c59998120dd852d060236dd0eef999fdff4557ed32fbbd0422d9805da249ee42482477513d719d409adccc2  Percona-XtraDB-Cluster-5.7.16-27.19.tar.gz
707c4b5e083326a173481764a275d2824fae4a88935227d38baa64d42312fdf1e6f7eae877c335c2744ea67c5fe128dafceed1f5af4dd0913a93b2f7269a00b9  fix-posix_timers.patch
42370de4d750e041bb5f2f9f167cf83d13b4904d99ccf1c83a116488906eb8bb7382b4ba64f8c6a09e8355c885ae00b2dff60a88db6f6d2bd868bdcf6da75ba8  fix-wsrep_sst_xtrabackup-v2.patch
da150182dd71f5839d4603c1294401ede2b72c3515ddcc8b48e38d830e779ffe71ccf3e93f626af31734e8487f4aee0347cbe1ec66484ea1500d7f87b4e1656e  percona-xtradb-cluster.cnf
ddee1e00059222744d3f0157b76aec0baf58011fdf394680c5ac86a0e41bf6293477c46a2c0f3b4c9e40001e7ec9b1af936e489dcaba2a81b832458d6a5bc5c4  mysqld.cnf
61a6ac4ad1b0a6742407b6d7b29149436a411a146e2fcb975ed473870bd0f621c64ca3799c4a10186270fd3f03e3cfc2ec42d1610e80e173a327b9f6af4a52aa  mysqld_safe.cnf
c2b2c6ad45b5b785d22e1ad07b21a58ce59cd28481e8933c51d20b7cd23407511a5764ad88d5b3874d4cd9270a92affca63908efe7990d52dce60a283cc81378  client.cnf
18562a455edaefd5870898b4bf7abdbf36b85953ad7877d5bac0f683e93193651c931ad31463f63d6f32289c1ce40cc49551ec89f027a4d9d00bf97b5e478f89  wsrep.cnf"
