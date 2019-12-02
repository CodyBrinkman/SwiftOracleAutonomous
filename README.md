# SwiftOracleAutonomous
Driver for Swift to Oracle Autonomous Database

This builds on top of https://github.com/goloveychuk/SwiftOracle and wraps ocilib (https://github.com/vrogier/ocilib)

Shoutout to Vincent Rogier (vrogier) himself for assistance on this project.

_This example uses an Ubuntu 18.04 instance_

1. Download Oracle Client (this example uses v19.3)
```
oracle-instantclinet*-basic-*.rpm
oracle-instantclinet*-devel-*.rpm
oracle-instantclinet*-sqlplus-*.rpm
```

2. Install RPMs
```
$ sudo alien -i oracle-instantclient*-basic-*.rpm
$ sudo alien -i oracle-instantclinet*-devel-*.rpm
$ sudo alien -i oracle-instantclinet*-sqlplus-*.rpm
```

3. Install libaio1
```
$ sudo apt install libaio1
```

4. Set environment variables
```
$ export ORACLE_HOME=/usr/lib/oracle/19.3/client64
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/oracle/19.3/client64/lib:/usr/local/lib
```

5. Download and configure OCILIB library
```
$ git clone https://github.com/vrogier/ocilib.git
$ tar -zxf ocilib-4.5.2-gnu.tar.gz
$ cd ocilib-4.5.2
$ ./configure --with-oracle-headers-path=/usr/include/oracle/19.3/client64/ --with-oracle-lib-path=/usr/lib/oracle/19.3/client64/lib CFLAGS="-O2 -m64"
$ make
$ sudo make install
```



```
