Steps to Build and Benchmark latest HHVM(mainline) on latest Fedora(27)

##### Please use latest GCC or Clang compiler to build latest HHVM as older version may not able to build due to cutting edge c++ feature inclusion.
 
### Install Dependency Packages

$sudo dnf install binutils-devel boost-devel bzip2-devel cmake3 composer cpp double-conversion-devel elfutils-libelf-devel enca expat-devel fastlz-devel fribidi fribidi-devel gcc-c++ git gitg gitg-libs gitk glib2-devel glog-devel gmp-devel gperf ImageMagick-devel jemalloc-devel kernel-tools libatomic libcap-devel libcurl-devel libdbi-dbd-pgsql libdwarf-devel libedit-devel libevent-devel libicu-devel libjpeg-turbo-devel libmcrypt-devel libmemcached-devel libpng-devel libvpx-devel libxml2-devel libxslt-devel libyaml-devel libzip-devel lz4-devel make mariadb mariadb-devel mariadb-server ncurses-compat-libs nginx numactl-devel ocaml ocaml-camomile-data ocaml-compiler-libs ocaml-findlib ocaml-findlib-devel ocaml-ocamlbuild ocaml-runtime ocaml-srpm-macros oniguruma-devel openldap-devel openssl-devel patch pcre-devel php-imap php-pgsql postgresql-devel psmisc re2 re2-devel readline-devel redhat-rpm-config siege sqlite-devel tbb-devel unixODBC-devel uw-imap uw-imap-devel

### Mitigate hhvm lz4 dependency by installing lz4 1.7.5 under /usr/local

This is needed as of now unless hhvm get migrated to newer version of lz4.

$wget https://github.com/lz4/lz4/archive/v1.7.5.tar.gz

$tar -zxvf v1.7.5.tar.gz

$cd lz4-1.7.5

$make

$sudo make install

### Get latest hhvm source

Get the latest HHVM source code from github. You may want to set http_proxy and https_proxy if you are in corporate proxy environment.

$git clone https://github.com/facebook/hhvm.git

$cd hhvm

$sed -i 's/git:/https:/' .gitmodules

$sed -i 's/git:/https:/' .git/config

$git submodule init

$git submodule update

$git submodule update --init --recursive

HHVM code should be fully downloaded after these steps. Now hhvm can be build with different compilers

### Building with latest GCC(7.x)

##### Configure

$/usr/bin/cmake -DMYSQL_UNIX_SOCK_ADDR=/dev/null -DCMAKE_INSTALL_PREFIX=${hhvminstallpath} -DCMAKE_C_COMPILER:FILEPATH=\`which gcc\` -DCMAKE_CXX_COMPILER:FILEPATH=\`which g++\`  -DCMAKE_AR=\`which ar\` -DCMAKE_VERBOSE_MAKEFILE=ON .

##### Make

$/usr/bin/make VERBOSE=1 SHELL="sh -x"

##### Install

$/usr/bin/make install

### Building with latest Clang(5.x)

##### Configure

$/usr/bin/cmake -DMYSQL_UNIX_SOCK_ADDR=/dev/null -DCLANG_FORCE_LIBSTDCXX=ON -DCMAKE_INSTALL_PREFIX=${hhvminstallpath} -DCMAKE_C_COMPILER:FILEPATH=\`which clang\` -DCMAKE_CXX_COMPILER:FILEPATH=\`which clang++\` -DCMAKE_VERBOSE_MAKEFILE=ON . 

##### Make

$/usr/bin/make VERBOSE=1 SHELL="sh -x" 

##### Install

$/usr/bin/make install

### Prepare machine for Oss-Performance Wordpress run

$echo 0 | sudo tee /proc/sys/kernel/randomize_va_space

$sudo /bin/cpupower frequency-set -g performance

$sudo systemctl disable abrtd

$sudo systemctl stop abrtd

$echo 1 | sudo tee /proc/sys/net/ipv4/tcp_tw_reuse

$sudo systemctl start nscd

$echo 3 | sudo tee /proc/sys/vm/drop_caches

$sudo systemctl stop pmcd

$sudo systemctl disable pmcd

$sudo systemctl stop pmlogger

$sudo systemctl disable pmlogger

$sudo systemctl stop libvirtd

$sudo systemctl disable libvirtd

$sudo systemctl stop xenconsoled

$sudo systemctl disable xenconsoled

$sudo systemctl stop xendomains

$sudo systemctl disable  xendomains

$sudo systemctl stop  vmtoolsd

$sudo systemctl disable  vmtoolsd

$sudo systemctl stop ModemManager

$sudo systemctl disable ModemManager

$sudo systemctl enable mariadb

$sudo systemctl start mariadb

### Install/Setup/Run OSS-Performance

##### Set HHVM PATH

$export PATH=${hhvminstallpath}/bin:$PATH

$export LD_LIBRARY_PATH=${hhvminstallpath}/lib64:${LD_LIBRARY_PATH}

##### If you are behind proxy

Find out your org proxy url and port say ${https_proxy_port}, it should look something like https://proxyserver.com:NNN

$git clone https://github.com/hhvm/oss-performance.git

$cd oss-performance

$curl -x ${https_proxy_port} -sS https://getcomposer.org/installer | php

$https_proxy=${https_proxy_port} hhvm composer.phar install

$export http_proxy=

$export https_proxy=



##### Kill any existing running hh_server

$killall hh_server

##### Fix MariaDB SQL

$cat sqlscript

drop database if exists wp_bench;

drop user if exists 'wp_bench'@'localhost';

create database wp_bench;

create user 'wp_bench'@'localhost' identified by 'wp_bench';

grant all privileges on wp_bench.* to wp_bench@localhost identified by 'wp_bench';

set global max_connections=2000;

flush privileges;


$mysql -u root < sqlscript

### Run Wordpress OSS Performance benchmark

$hhvm -d 'hhvm.php7.all=0' perf.php --wordpress --hhvm=\`which hhvm\`

##### If successful, it should produce output like

** There is no tearDownTest

{

    "Combined": {

        "Siege requests": 152335,

        "Siege wall sec": 0.08,

        "Siege RPS": 2542.31,

        "Siege successful requests": 145135,

        "Siege failed requests": 0,

        "Nginx hits": 145935,

        "Nginx avg bytes": 30020.103265152,

        "Nginx avg time": 0.078105800527626,

        "Nginx P50 time": 0.079,

        "Nginx P90 time": 0.094,

        "Nginx P95 time": 0.099,

        "Nginx P99 time": 0.109,

        "Nginx 200": 132015,

        "Nginx 499": 120,

        "Nginx 301": 6600,

        "Nginx 404": 7200,

        "canonical": 1

    }

}

### How to interpret result

First look at "Siege failed requests". Result is not valid if it is NOT 0.

Look for "Siege RPS" number. Higher is better.

