## How to build GCC 4.8.5 on Ubuntu 16.04 LTS

This issue appear when you try to build gcc which is older than system default gcc.

In this case Ubuntu 16.04 LTS and one try gcc build is 4.8.5.

GCC Build command

tar -jxvf gcc-4.8.5.tar.bz2

cd gcc-4.8.5

cd ..

mkdir build

cd build

#### If everything go good, your get the build OR you may get error message something like this.

make[5]: Entering directory '/local/foo/build/x86_64-linux-gnu/libstdc++-v3/po'

msgfmt -o de.mo ../../../../gcc-4.8.5/libstdc++-v3/po/de.po

msgfmt: /local/foo/build/x86_64-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6: version `CXXABI_1.3.8' not found (required by /usr/lib/x86_64-linux-gnu/libicuuc.so.55)

Makefile:460: recipe for target 'de.mo' failed

make[5]: *** [de.mo] Error 1

#### Problem caused by uincompatibilty between system libstdc++.so.6 and gcc build /local/foo/build/x86_64-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6

msgfmt expecting system libstdc++.so.6 but gcc build providing gcc built local/foo/build/x86_64-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6

#### Lets dig little further.

$ ldd /usr/bin/msgfmt | grep libstdc
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007ffb6c1c3000)
        
$ strings /usr/lib/x86_64-linux-gnu/libstdc++.so.6 | grep CXXABI_

CXXABI_1.3

CXXABI_1.3.1

CXXABI_1.3.2

CXXABI_1.3.3

CXXABI_1.3.4

CXXABI_1.3.5

CXXABI_1.3.6

CXXABI_1.3.7

CXXABI_1.3.8

CXXABI_1.3.9

CXXABI_TM_1

CXXABI_FLOAT128

$ strings /local/foo/build/x86_64-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6 | grep CXXABI_

CXXABI_1.3

CXXABI_1.3.1

CXXABI_1.3.2

CXXABI_1.3.3

CXXABI_1.3.4

CXXABI_1.3.5

CXXABI_1.3.6

CXXABI_1.3.7

CXXABI_TM_1

CXXABI_1.3

CXXABI_1.3.2

CXXABI_1.3.6

CXXABI_1.3.1

CXXABI_1.3.5

CXXABI_1.3.4

CXXABI_TM_1

CXXABI_1.3.7

CXXABI_1.3.3

As you can see GCC built libstdc++.so.6 doesn't have support for CXXABI_1.3.8, needed for msgfmt. Issue here, gcc build sets LD_LIBRARY_PATH and prepend gcc build libstdc++.so.6 before system libstdc++.so.6. Since library name(libstdc++.so.6) is exactly same it pickup gcc built one.

This happens when gcc version under build is lower than system default gcc. To get around this problem, just create a wrapper for the affected tool i.e.

$cat /local/bin/msgfmt

export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH

tool_wrap=`echo $0 |  sed -e 's/^.*\///g'`

exec /usr/bin/${tool_wrap} ${1+"$@"}

#### Now set the path before invoking gcc build like.

export PATH=/local/bin:$PATH

Now gcc build will go though as every time msgfmt called it's dependency get satisfied with system libstdc++.so.6 as intended.

#### Question/Comments: 

  Leave a message about your experience.










