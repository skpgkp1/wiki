### How to build and use cpurt library. This can help performance wise if server processor is newer but glibc is old. 

#### checkout glibc source and build

$ mkdir /local/foo/cpurt

$ cd /local/foo/cpurt

$ git clone https://github.com/hjl-tools/glibc

$ cd glibc

$ git checkout hjl/cpu-rt/master

$ mkdir ../build ../binary

$ cd ../build

$ ../glibc/configure --enable-hardcoded-path-in-tests --enable-cpu-rt --prefix=/local/foo/cpurt/cpurtinstall

$ make -j32

$ make check

$ make install # It should put all the library in cpurtinstall diectory.

#### How to use newly build cpurt library.

$ export LD_PRELOAD=/local/foo/cpurt/cpurtinstall/lib/libcpu-rt-c.so

#### Run application say foo.out, it should use latest libcpurt library.

$ ./foo.out

