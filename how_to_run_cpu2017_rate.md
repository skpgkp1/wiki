### How to run spec CPU2017 for ```rate```.

#### Understanding and running cpu benchmark is quite complex to grasp. Here is my simple setup to run cpu CPU2017 run with gcc.

Assuming you have valid copy of CPU2017 and gcc compiler to test.

CPU 2017 Install Directory: ```/local/foo/cpu2017```

GCC Install Directory: ```/local/foo/comp/gcc```


$ cd /local/foo/cpu2017
```
$ ./spec2017.rate
runcpu v5038 - Copyright 1999-2016 Standard Performance Evaluation Corporation
Using 'linux-x86_64' tools
Reading file manifests... read 32125 entries from 2 files in 0.07s (433682 files/s)
Loading runcpu modules................
Locating benchmarks...found 47 benchmarks in 53 benchsets.
Reading config file '/local/foo/cpu2017/config/spec2017.cfg'
2 configurations selected:

Action   Benchmarks
------   ----------------------------------------------------------------------
validate intrate
validate fprate
-------------------------------------------------------------------------------

Setting up environment for running intrate...
Starting runcpu for intrate...
Running "specperl /local/foo/cpu2017/Docs/sysinfo" to gather system information.
sysinfo: r5007 of 2016-11-15 (fc8dc82f217779bedfed4d694d580ba9)
sysinfo: Getting system information for Linux...
sysinfo: ...getting CPU info
sysinfo: ...getting info from numactl
sysinfo: ...getting memory info
sysinfo: ...getting OS info
sysinfo: ...getting disk info
sysinfo: ...trying to get DIMM info from dmidecode
Benchmarks selected: 500.perlbench_r, 502.gcc_r, 505.mcf_r, 520.omnetpp_r, 523.xalancbmk_r, 525.x264_r, 531.deepsjeng_r, 541.leela_r, 548.exchange2_r, 557.xz_r, 999.specrand_ir
Compiling Binaries
  Building 500.perlbench_r base 201808313264: (build_base_201808313264.0000) [2018-08-31 13:56:02]
  Building 502.gcc_r base 201808313264: (build_base_201808313264.0000) [2018-08-31 13:56:19]
  Building 505.mcf_r base 201808313264: (build_base_201808313264.0000) [2018-08-31 13:57:41]
  Building 520.omnetpp_r base 201808313264: (build_base_201808313264.0000) [2018-08-31 13:57:42]
```


```

$ cat spec2017.rate
. ./shrc
ulimit -s unlimited
export NUM="3"
export bits="64"
export GCC_DIR=/local/foo/comp/gcc
export corecount=`lscpu | grep '^CPU(s):' | cut -f2 -d':' | sed -e 's/ //g'`
export BASEOPT="-march=native -mfpmath=sse -O2" # These compiler option will be used for all benchmarks.
export SPECDIR="/local/foo/cpu2017"
cd $SPECDIR
# Manually Clean spec directory
rm -rf $SPECDIR/benchspec/CPU/*/run
rm -rf $SPECDIR/benchspec/CPU/*/build
rm -rf $SPECDIR/benchspec/CPU/*/exe

# Build and run the benchmark
runcpu  -a validate --loose -I -i ref -o asc -c spec2017.cfg --label `date +%Y%m%d$$` -T base -n $NUM  --copies $corecount -I specrate

```


```

$ cat spec2017.cfg
#--------- Preprocessor -------------------------------------------------------
#
# PATH - Please edit this:
%define  gcc_dir %{ENV_GCC_DIR}

# Optionally edit if you wish:
%define  bits         %{ENV_bits}      # set this to either 32 or 64
%define  build_ncpus %{ENV_corecount}      # controls number of simultaneous compiles

# You should not need to edit these
%if %{bits} == 64
%   define model -m64
%elif %{bits} == 32
%   define model -m32
%else
%   error wait, how many bits?
%endif
%define  os          LINUX

#--------- Global Settings ----------------------------------------------------
# For info, see:
#            https://www.spec.org/auto/cpu2017/Docs/config.html#fieldname   XXX
#   Example: https://www.spec.org/auto/cpu2017/Docs/config.html#tune

command_add_redirect = 1
env_vars             = 1
ignore_errors        = 1
allow_label_override = yes
iterations           = 1
line_width           = 1020
log_line_width       = 1020
makeflags            = --jobs=%{build_ncpus} --load-average=%{build_ncpus}
output_format        = txt,cfg,pdf,csv
preenv               = 1
preENV_OMP_STACKSIZE = 120M
tune                 = base

#--------- Compilers ----------------------------------------------------------
default:
   preENV_LD_LIBRARY_PATH  = %{gcc_dir}/lib64/:%{gcc_dir}/lib
   SPECLANG                = %{gcc_dir}/bin/
   CC                      = $(SPECLANG)gcc %{model}
   CXX                     = $(SPECLANG)g++ %{model}
   FC                      = $(SPECLANG)gfortran %{model}
   # How to say "Show me your version, please"
   CC_VERSION_OPTION       = -v
   CXX_VERSION_OPTION      = -v
   FC_VERSION_OPTION       = -v

#--------- Portability --------------------------------------------------------
default:
# Data model
%if %{bits} == 32
    EXTRA_PORTABILITY = -D_FILE_OFFSET_BITS=64
%else
    EXTRA_PORTABILITY = -DSPEC_LP64
%endif


# Benchmark-specific portability (ordered by last 2 digits of bmark number)

500.perlbench_r,600.perlbench_s:  #lang='C'
%if %{bits} == 32
%   define suffix IA32
%else
%   define suffix X64
%endif
   PORTABILITY    = -DSPEC_%{os}_%{suffix}

502.gcc_r,602.gcc_s:  #lang='C'
%if %{bits} == 32
    PORTABILITY = -fgnu89-inline
%else
    PORTABILITY = -z muldefs -m64
%endif

521.wrf_r,621.wrf_s:  #lang='F,C'
   CPORTABILITY  = -DSPEC_CASE_FLAG -DSPEC_LINUX
   FPORTABILITY  = -fconvert=big-endian

523.xalancbmk_r,623.xalancbmk_s:  #lang='CXX'
   PORTABILITY   = -DSPEC_%{os}

526.blender_r:  #lang='CXX,C'
   PORTABILITY   = -funsigned-char -DSPEC_LINUX

527.cam4_r,627.cam4_s:  #lang='F,C'
   PORTABILITY   = -DSPEC_CASE_FLAG

628.pop2_s:  #lang='F,C'
   PORTABILITY   = -DSPEC_CASE_FLAG -fconvert=big-endian




#--------  Baseline Tuning Flags ----------------------------------------------

specspeed:
   EXTRA_OPTIMIZE = -fopenmp -DSPEC_OPENMP

default=base:
   OPTIMIZE       = -g -fno-strict-aliasing -fno-unsafe-math-optimizations   %{ENV_BASEOPT}
   CXXOPTIMIZE = -std=c++03

#--------- Your info ---------------------------------------------------------
# To understand the difference between hw_vendor/sponsor/tester, see:
#     www.spec.org/auto/cpu2017/Docs/config.html#test_sponsor
intrate,intspeed,fprate,fpspeed: # Important: keep this line
   hw_vendor          = My Corporation
   tester             = My Corporation
   test_sponsor       = My Corporation
   license_num        = nnn (Your SPEC license number)

#--------- Fill out this section ----------------------------------------------
intrate,intspeed,fprate,fpspeed: # Important: keep this line
                        # Example                             # Brief info about field
   hw_avail           = # Nov-2099                            # Date of LAST hardware component to ship
   sw_avail           = # Nov-2099                            # Date of LAST software component to ship
   hw_cpu_nominal_mhz = # 9999                                # Nominal chip frequency, in MHz
   hw_cpu_max_mhz     = # 9999                                # Max chip frequency, in MHz
   hw_ncores          = # 9999                                # number cores enabled
   hw_nthreadspercore = # 9                                   # number threads enabled per core
   hw_ncpuorder       = # 1-9 chips                           # Ordering options

   hw_model           = # TurboBlaster 3000                   # system model name
   hw_other           = # TurboNUMA Router 10 Gb              # Other perf-relevant hw, or "None"
   sw_other           = # TurboHeap Library V8.1              # Other perf-relevant sw, or "None"

   hw_pcache          = # 99 KB I + 99 KB D on chip per core  # Primary cache size, type, location
   hw_scache          = # 99 KB I+D on chip per 9 cores       # Second cache or "None"
   hw_tcache          = # 9 MB I+D on chip per chip           # Third  cache or "None"
   hw_ocache          = # 9 GB I+D off chip per system board  # Other cache or "None"

   hw_memory001       = # 4 TB (256 x 16 GB 2Rx4 PC4-2133P-R, # N GB (M x N GB nRxn
   hw_memory002       = # running at 1600 MHz)                # PCn-nnnnnR-n[, ECC and other info])

#--------- Sysinfo fields - You may need to adjust this section ---------------
# The following are partly filled out by sysinfo.
#       www.spec.org/auto/cpu2017/Docs/config.html#sysinfo
# Uncomment lines for which you already have a better answer than sysinfo
#
intrate,intspeed,fprate,fpspeed: # Important: keep this line
                        # Example               # Brief info about field
 # hw_cpu_name        = # Intel Xeon E9-9999 v9               # chip name
 # hw_disk            = # 9 x 9 TB SATA III 9999 RPM          # Size, type, other perf-relevant info
 # hw_nchips          = # 99                                  # number chips enabled
 # sw_file            = # ext99                               # File system
 # sw_state           = # Run level 99                        # Software state.

 # sw_os001           = # Linux Sailboat                      # Operating system
 # sw_os002           = # Distribution 7.2 SP1                # and version
 # prepared_by        = # Ima Sailor                          # Whatever you like: is never output


__HASH__
```
