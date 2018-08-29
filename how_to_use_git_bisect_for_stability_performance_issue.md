## How to effectively use git bisect feature to find guilty change set for stability and performance regression for any git project(in this case hhvm used as an example)

First find Good and Bad git sha1 commit id to narrow down scope of git bisect

Bad git id: 4b20caf22d16ef930be847716d2836290816a25e

Good git id: 45ca481a56e94c12a085eeeb7ac62d56761319c9

### Checkout the hhvm mainline code:

$mkdir /local/foo/hhvm/exp

$git clone https://github.com/facebook/hhvm.git hhvm-mainline

$cd hhvm-mainline

$git log

### Set the bound for git bisect

$git bisect start `4b20caf22d16ef930be847716d2836290816a25e`  `45ca481a56e94c12a085eeeb7ac62d56761319c9` --

### Create a script which can define good and bad case.

Good case return 0 and bad case retrun non-zero.

$git bisect run /local/foo/hhvm_perf_triage.sh > /tmp/log 2>/&1

```

$cat /local/foo/hhvm_perf_triage.sh
export comp="5.5.0" && \
export PATH=/local/foo/binutilsbin/bin:/usr/local/bin:$PATH \
&& cd /local/foo/hhvm/exp/hhvm-mainline && \
git submodule init && \
git submodule update && \
git submodule update --init --recursive && \
/usr/bin/cmake -DMYSQL_UNIX_SOCK_ADDR=/dev/null -DCMAKE_INSTALL_PREFIX=/local/foo/hhvm/hhvmbin${comp} \
-DCMAKE_C_COMPILER:FILEPATH=/local/foo/hhvm/${comp}/gcc \
-DCMAKE_CXX_COMPILER:FILEPATH=/local/foo/hhvm/${comp}/g++ \
-DCMAKE_VERBOSE_MAKEFILE=ON . && \
/usr/bin/make VERBOSE=1 SHELL="sh -x" -j80 && \
/usr/bin/make install && \
cd /local/foo/hhvm/oss-performance && \
export PATH=/local/foo/hhvm/hhvmbin${comp}/bin:$PATH && \
export LD_LIBRARY_PATH=/local/foo/hhvm/hhvmbin${comp}/lib64:$LD_LIBRARY_PATH && \
unset http_proxy && unset https_proxy && \
hhvm  perf.php --wordpress --hhvm=`which hhvm` > perf.log.hhvmbin${comp} 2>&1 && \
export result=`grep RPS /local/foo/hhvm/oss-performance/perf.log.hhvmbin${comp} | \
cut -f2 -d':' | cut -f1 -d'.' | sed -e 's/ //g'` && echo "RPS:$result" && \
if [ $result -lt 2600 ]; then      echo "Bad:$result"; exit 7; else      echo "Good:$result";exit 0; fi

```
