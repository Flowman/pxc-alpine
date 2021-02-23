# Contributor: Peter Szalatnay <theotherland@gmail.com>
# Maintainer: Peter Szalatnay <theotherland@gmail.com>
pkgname=percona-xtradb-cluster
pkgver=8.0.21
_pkgver="$pkgver-12.1"
pkgrel=0
pkgdesc="Percona XtraDB Cluster is an active/active high availability and high scalability open source solution for MySQL clustering"
url="https://www.percona.com/software/mysql-database/percona-xtradb-cluster"
pkgusers="mysql"
pkggroups="mysql"
arch="all"
license="GPL2"
replaces="mariadb"
depends="
		$pkgname-common
		socat
		iproute2
		procps
		findutils
		coreutils
		tzdata
		bash
		"
makedepends="
		scons
		check-dev
		boost-dev
		openssl-dev
		openldap-dev
		cmake
		linux-headers
		numactl-dev
		readline-dev
		libtirpc-dev
		rpcgen
		curl-dev
		linux-pam-dev
		bison
		"
install="$pkgname.pre-install"
subpackages="
		$pkgname-doc
		$pkgname-dev
		$pkgname-common
		$pkgname-client
		$pkgname-router
		$pkgname-test:mytest
		"
source="http://dev.alpinelinux.org/archive/$pkgname/$pkgname-$_pkgver.tar.gz
		fix-gu_arch.patch
		fix-gu_alloc.patch
		fix-linux_syscall_support.patch
		fix-connection.patch
		fix-udf_utils.patch
		fix-signal_handler.patch
		"

_giturl="https://github.com/percona/percona-xtradb-cluster.git"
_gittag="Percona-XtraDB-Cluster-$_pkgver"

builddir="$srcdir/$pkgname"

snapshot() {
	mkdir -p "$srcdir"
	cd "${SRCDEST:-$srcdir}"
	if ! [ -d $pkgname ]; then
		git clone --branch $_gittag --depth 1 $_giturl
		cd $pkgname
		git submodule init
		git submodule update
		git submodule foreach --recursive 'git submodule init && git submodule update'
	else
		cd $pkgname
		git fetch
		git submodule init
		git submodule update
		git submodule foreach --recursive 'git submodule init && git submodule update'
	fi

	cd "$SRCDEST"
	tar --exclude-vcs -zcf $pkgname-$_pkgver.tar.gz ./$pkgname

	#scp "$SRCDEST"/$pkgname-$_pkgver.tar.gz dev.alpinelinux.org:/archive/$pkgname/
}

# Notes:
# PXC require boost 1.72 or it will now compile, alpine never have the right version required to build
build() {
	cd "$builddir"/percona-xtradb-cluster-galera

	local GALERA_REVISION="$(test -r GALERA-REVISION && cat GALERA-REVISION)"

	scons psi=1 --config=force revno="$GALERA_REVISION" libgalera_smm.so
	scons --config=force revno="$GALERA_REVISION" garb/garbd

	cd "$builddir"

	source "VERSION"
	local MYSQL_VERSION="$MYSQL_VERSION_MAJOR.$MYSQL_VERSION_MINOR.$MYSQL_VERSION_PATCH"
	local PERCONA_SERVER_EXTENSION="$(echo $MYSQL_VERSION_EXTRA | sed 's/^-/rel/')"
	local TAG='1'

	WSREP_VERSION="$(grep WSREP_INTERFACE_VERSION wsrep-lib/wsrep-API/v26/wsrep_api.h | cut -d '"' -f2).$(grep 'SET(WSREP_PATCH_VERSION'  "cmake/wsrep-lib.cmake" | cut -d '"' -f2)"

	local COMMENT="Percona XtraDB Cluster binary (GPL) $MYSQL_VERSION, WSREP version $WSREP_VERSION"

	CFLAGS="$CFLAGS -DPERCONA_INNODB_VERSION=$PERCONA_SERVER_EXTENSION"
	CXXFLAGS="$CXXFLAGS -DPERCONA_INNODB_VERSION=$PERCONA_SERVER_EXTENSION"

	cmake . -DBUILD_CONFIG=mysql_release \
			-DCMAKE_BUILD_TYPE=RelWithDebInfo \
			-DFEATURE_SET=community \
			-DSYSCONFDIR=/etc/mysql \
			-DCMAKE_INSTALL_PREFIX=/usr \
			-DMYSQL_DATADIR=/var/lib/mysql \
			-DROUTER_INSTALL_DATADIR=/var/lib/mysqlrouter \
			-DROUTER_INSTALL_LIBDIR=lib/mysqlrouter/private \
			-DROUTER_INSTALL_PLUGINDIR=lib/mysqlrouter/plugin \
			-DINSTALL_INFODIR=share/mysql/docs \
			-DINSTALL_MANDIR=share/man \
			-DINSTALL_PLUGINDIR=lib/mysql/plugin \
			-DINSTALL_INCLUDEDIR=include/mysql \
			-DINSTALL_DOCREADMEDIR=share/doc/mysql \
			-DINSTALL_SUPPORTFILESDIR=share/mysql \
			-DINSTALL_MYSQLSHAREDIR=share/mysql \
			-DINSTALL_DOCDIR=share/mysql/docs \
			-DINSTALL_SHAREDIR=share/mysql \
			-DMYSQL_UNIX_ADDR=/run/mysqld/mysqld.sock \
			-DCOMPILATION_COMMENT="$COMMENT" \
			-DWITH_PAM=ON \
			-DWITHOUT_ROCKSDB=ON \
			-DWITHOUT_TOKUDB=ON \
			-DWITH_INNODB_MEMCACHED=ON \
			-DDOWNLOAD_BOOST=ON \
			-DWITH_BOOST="$builddir"/libboost \
			-DFORCE_INSOURCE_BUILD=ON \
			-DWITH_SYSTEM_LIBS=ON \
			-DWITH_PROTOBUF=bundled \
			-DWITH_RAPIDJSON=bundled \
			-DWITH_ICU=bundled \
			-DWITH_LZ4=bundled \
			-DWITH_EDITLINE=bundled \
			-DWITH_LIBEVENT=bundled \
			-DWITH_ZSTD=bundled \
			-DWITH_NUMA=ON \
			-DWITH_LDAP=system \
			-DMYSQL_SERVER_SUFFIX=".$TAG" \
			-DWITH_WSREP=ON \
			-DWITH_UNIT_TESTS=OFF \

	# print config options to log
	cmake -L

	make
}

package() {
	depends="percona-xtrabackup"

	make DESTDIR="$pkgdir/" install

	install -Dm755 "$pkgdir"/usr/share/mysql/mysql.server \
		"$pkgdir"/etc/init.d/mysql
	install -Dm644 "$pkgdir"/usr/share/mysql/mysql-log-rotate \
		"$pkgdir"/etc/logrotate.d/mysql

	install -Dm644 "$builddir"/COPYING.innodb-deadlock-count-patch \
		"$pkgdir"/usr/share/licenses/$pkgname/COPYING.innodb-deadlock-count-patch
	install -Dm644 "$builddir"/COPYING.show_temp_51 \
		"$pkgdir"/usr/share/licenses/$pkgname/COPYING.show_temp_51
	mv "$pkgdir"/usr/LICENSE.router \
		"$pkgdir"/usr/README.router \
		"$pkgdir"/usr/share/licenses/$pkgname/

	install -Dm755 "$builddir"/percona-xtradb-cluster-galera/libgalera_smm.so \
		"$pkgdir"/usr/lib/galera4/libgalera_smm.so
	install -Dm755 "$builddir"/percona-xtradb-cluster-galera/garb/garbd \
		"$pkgdir"/usr/bin/garbd

	install -Dm640 -o mysql "$builddir"/build-ps/debian/extra/mysql.cnf \
		"$pkgdir"/etc/mysql/mysql.cnf
	install -Dm640 -o mysql "$builddir"/build-ps/debian/extra/mysql.cnf \
		"$pkgdir"/etc/mysql/my.cnf
	install -Dm640 -o mysql "$builddir"/build-ps/debian/extra/my.cnf.fallback \
		"$pkgdir"/etc/mysql/my.cnf.fallback

	mkdir -p "$pkgdir"/etc/mysql/mysql.conf.d/ \
		"$pkgdir"/etc/mysql/conf.d/

	install -Dm640 -o mysql "$builddir"/build-ps/debian/extra/conf.d/mysql.cnf \
		"$pkgdir"/etc/mysql/conf.d/mysql.cnf
	install -Dm640 -o mysql "$builddir"/build-ps/debian/extra/mysqld.cnf \
		"$pkgdir"/etc/mysql/mysql.conf.d/mysqld.cnf

	# fix mysqld.cnf
	sed -i "s/socket=\/var\/run\/mysqld/socket=\/run\/mysqld/g" "$pkgdir"/etc/mysql/mysql.conf.d/mysqld.cnf
	sed -i "s/pid-file=\/var\/run\/mysqld/pid-file=\/run\/mysqld/g" "$pkgdir"/etc/mysql/mysql.conf.d/mysqld.cnf

	# fix init.d script
	sed -i "s/. \/etc\/rc.d\/init.d\/functions/#. \/etc\/rc.d\/init.d\/functions/g" "$pkgdir"/usr/share/mysql/mysql.server
	sed -i "s/action \$\"Initializing MySQL database: \" \/usr\/sbin\/mysqld --initialize --datadir=\"\$datadir\" --user=mysql/\/usr\/bin\/mysqld --initialize --datadir=\"\$datadir\" --user=mysql/g" "$pkgdir"/usr/share/mysql/mysql.server

	# Remove files that exists in xtrabackup package and will conflict
	rm \
		"$pkgdir"/usr/lib/private/libprotobuf-lite.so.3.11.4 \
		"$pkgdir"/usr/lib/private/libprotobuf.so.3.11.4

	# Remove files we explicitly do not want to package, avoids 'unpackaged files' warning.
	rm -rf \
		"$pkgdir"/usr/var/ \
		"$pkgdir"/usr/lib/private \
		"$pkgdir"/usr/bin/ps-admin \
		"$pkgdir"/usr/lib/libperconaserverclient.so*
}

dev() {
	default_dev
	replaces="libmysqlclient mysql-dev"
	provides="mysql-dev=$pkgver-r$pkgrel"
	local bins="mysql_config mysql_config_editor"
	mkdir -p "$subpkgdir"/usr/bin
	for i in $bins; do
		mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/
	done
}

common() {
	pkgdesc="Percona XtraDB Cluster common files for both server and client"
	replaces="mysql-common mariadb-common"
	depends=
	install -Dm640 -o mysql /dev/null "$subpkgdir"/var/log/mysqld.log
	install -Dm750 -o mysql -d "$subpkgdir"/var/log/mysql
	install -Dm750 -o mysql -d "$subpkgdir"/usr/lib/mysql
	install -Dm750 -o mysql -d "$subpkgdir"/run/mysqld
	mkdir -p "$subpkgdir"/usr/share/mysql \
		"$subpkgdir"/etc
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

mytest() {
	pkgdesc="The test suite distributed with Percona XtraDB Cluster"
	mkdir -p "$subpkgdir"/usr/bin
	mv "$pkgdir"/usr/bin/mysql_client_test \
		"$pkgdir"/usr/bin/mysqlxtest \
		"$pkgdir"/usr/bin/mysqltest \
		"$pkgdir"/usr/bin/mysqltest_safe_process \
		"$subpkgdir"/usr/bin/
	mv "$pkgdir"/usr/mysql-test \
		"$subpkgdir"/usr/
}

client() {
	pkgdesc="client for the Percona XtraDB Cluster database"
	depends="$pkgname-common"
	replaces="mysql-client mariadb-client"
	mkdir -p "$subpkgdir"/usr/bin/
	local bins="myisam_ftdump mysql mysqladmin
		mysqlcheck mysqldump mysqldumpslow
		mysqlimport mysqlpump mysqlshow mysqlslap"
	for i in $bins; do
		mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/
	done
}

router() {
	pkgdesc="router for the Percona XtraDB Cluster database"
	depends="$pkgname"
	mkdir -p "$subpkgdir"/usr/lib/mysqlrouter/plugin/ \
		"$subpkgdir"/usr/lib/mysqlrouter/private/ \
		"$subpkgdir"/etc/mysqlrouter/
	install -Dm640 -o mysql "$builddir"/build-ps/debian/extra/mysqlrouter.conf \
		"$subpkgdir"/etc/mysqlrouter/mysqlrouter.conf
	mv "$pkgdir"/usr/lib/mysqlrouter/plugin/*.so* \
		"$subpkgdir"/usr/lib/mysqlrouter/plugin/
	mv "$pkgdir"/usr/lib/mysqlrouter/private/libmysql*.so.1 \
		"$subpkgdir"/usr/lib/mysqlrouter/private
	mv "$pkgdir"/usr/lib/mysqlrouter/private/libprotobuf-lite.so.* \
		"$subpkgdir"/usr/lib/mysqlrouter/private
	local bins="mysqlrouter mysqlrouter_keyring mysqlrouter_plugin_info
		mysqlrouter_passwd"
	mkdir -p "$subpkgdir"/usr/bin/
	for i in $bins; do
		mv "$pkgdir"/usr/bin/${i} "$subpkgdir"/usr/bin/
	done
	rm -rf \
		"$pkgdir"/usr/lib/mysqlrouter \
		"$pkgdir"/var/lib/mysqlrouter \
		"$pkgdir"/usr/share/mysql/docs/sample_mysqlrouter.conf
}
