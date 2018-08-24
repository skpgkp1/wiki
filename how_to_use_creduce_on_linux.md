## How to effectively use creduce to cutdown size of source file reproducer.

### First build/install latest version of creduce on machine. Old version may not have parallel creduce functionality.

https://embed.cs.utah.edu/creduce/

Get the preprocessed file say server.cpp.i

### Find the good case command line:

Good: `/local/foo/good/g++ -g -O2 -pthread -ftemplate-depth-256 -c -o server.cpp.o server.cpp.i`

### Find the bad case command line:

Bad:  `/local/foo/bad/g++ -g -O2 -pthread -ftemplate-depth-256 -c -o server.cpp.o server.cpp.i`

Let say bad case produce output like

`"error: ambiguous overload for foo"`

Goal is to cutdown server.cpp.i content to exactly produce one install of error "error: ambiguous overload for foo".

### Create a shell script say test.sh like.

$ cat test.sh
/usr/bin/g++ -g -O2 -pthread -ftemplate-depth-256 -c -o server.cpp.o server.cpp.i && \
! g++ -g -O2 -pthread -ftemplate-depth-256 -c -o server.cpp.o server.cpp.i > log 2>&1 && \
export pp=\``grep ' error:' log | wc -l`\` &&  export tt=\``grep ' error: ambiguous overload for ' log | wc -l`\`  && \
if [ $tt -eq 1 ] && [ $pp -eq 1 ] ; then     exit 0; else     exit 1; fi

### Run creduce:

$ creduce ./test.sh server.cpp.i --n 48

where 48 is number of instance of creduce you want to run in parallel. Adjust this number according to your machine total number of core.

Some time it take very long time for advance c++ program. If successfull it will produce cut down version of server.cpp.i file.
