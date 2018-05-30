## How to build and benchmark Mariadb with sysbench.

### Get the latest mariadb source code from github
    
    $mkdir /local/foo
    $git clone https://github.com/MariaDB/server.git mariadb
    $cd mariadb
    $cmake  -DCMAKE_INSTALL_PREFIX=/local/foo/mariadbbin -DCMAKE_C_COMPILER:FILEPATH=`which gcc` \
    -DCMAKE_CXX_COMPILER:FILEPATH=`which g++`  -DBUILD_CONFIG=mysql_release -DWITH_JEMALLOC=yes .
    $/usr/bin/make VERBOSE=1 -j32
    $/usr/bin/make install
    
### Create configuration file

    $cd /local/foo/mariadbbin
    Create configuration file like.
    $cat my.cnf
    [mysqld]
    user=user1
    datadir=/local/foo/mariadbbin/data
    port=3307
    socket=/local/foo/mariadbbin/tmp/mysql.sock

    [mysqld_safe]
    log-error=/local/foo/mariadbbin/log/mysqld.log
    pid-file=/local/foo/mariadbbin/mysqld.pid

    [client]
    port=3307
    user=user1
    socket=/local/foo/mariadbbin/tmp/mysql.sock

    [mysqladmin]
    user=root
    port=3307
    socket=/local/foo/mariadbbin/tmp/mysql.sock

    [user1]
    port=3307
    socket=/local/foo/mariadbbin/tmp/mysql.sock

    [mysql_install_db]
    user=user1
    port=3307
    basedir=
    datadir=/local/foo/mariadbbin/data
    socket=/local/foo/mariadbbin/tmp/mysql.sock

### Mariadb install database

    $cd /local/foo/mariadbbin
    $scripts/mysql_install_db --user=user1 --basedir=/local/foo/mariadbbin --defaults-file=/local/foo/mariadbbin/my.cnf
    Installing MariaDB/MySQL system tables in '/local/foo/mariadbbin/data' ...
    OK

    To start mysqld at boot time you have to copy
    support-files/mysql.server to the right place for your system

    PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
    To do so, start the server, then issue the following commands:

    '/local/foo/mariadbbin/bin/mysqladmin' -u root password 'new-password'
    '/local/foo/mariadbbin/bin/mysqladmin' -u root -h hostserver password 'new-password'

    Alternatively you can run:
    '/local/foo/mariadbbin/bin/mysql_secure_installation'

    which will also give you the option of removing the test
    databases and anonymous user created by default.  This is
    strongly recommended for production servers.

    See the MariaDB Knowledgebase at http://mariadb.com/kb or the
    MySQL manual for more instructions.

    You can start the MariaDB daemon with:
    cd '/local/foo/mariadbbin' ; /local/foo/mariadbbin/bin/mysqld_safe --datadir='/local/foo/mariadbbin/data'

    You can test the MariaDB daemon with mysql-test-run.pl
    cd '/local/foo/mariadbbin/mysql-test' ; perl mysql-test-run.pl

    Please report any problems at http://mariadb.org/jira

    The latest information about MariaDB is available at http://mariadb.org/.
    You can find additional information about the MySQL part at:
    http://dev.mysql.com
    Consider joining MariaDB's strong and vibrant community:
    https://mariadb.org/get-involved/

### Setup and start Mariadb server
    $cd /local/foo/mariadbbin
    $mkdir log tmp
    $export PATH=/local/foo/mariadbbin/bin:${PATH}
    $export LD_LIBRARY_PATH=/local/foo/mariadbbin/lib:${LD_LIBRARY_PATH}
    $bin/mysqld_safe --defaults-file=my.cnf --no-auto-restart
    180530 16:08:49 mysqld_safe Logging to '/local/foo/mariadbbin/log/mysqld.log'.
    180530 16:08:49 mysqld_safe Starting mysqld daemon with databases from /local/foo/mariadbbin/data
    $sleep 5

### Sysbench read test

    $cd /local/foo/mariadbbin
    $export TEST_DIR=/usr/share/sysbench
    $bin/mysqladmin -u root --socket=/local/foo/mariadbbin/tmp/mysql.sock drop sbtest -f
    $bin/mysqladmin -u root --socket=/local/foo/mariadbbin/tmp/mysql.sock create sbtest
    $sysbench ${TEST_DIR}/oltp_read_only.lua --mysql-socket=/local/foo/mariadbbin/tmp/mysql.sock \
    --mysql-user=root --db-driver=mysql --time=60  --threads=48 prepare
    $sysbench ${TEST_DIR}/oltp_read_only.lua --mysql-socket=/local/foo/mariadbbin/tmp/mysql.sock \
    --mysql-user=root --db-driver=mysql --time=60 --threads=48 run

   



    
    

    
