### How to debug cpu2017 performance degradation to file level. It's important to understand, which file in order to create test case or file bug report.

#### This script try to triage faulty file using binary search methodology if possible. In this example, it trying to find performance effect of compiler option -foption.

```
$ how_to_triage_cpu2017_file_for_perf.sh 505 spec2017.cfg /local/foo/comp/gcc-9.0.0 "-march=native -mfpmath=sse -foption" "-foption=none"
Intital value: 270.19
Range: 1 6
New Value1: 266.47
Range: 7 12
New Value2: 252.50
Range: 7 9
New Value1: 267.68
Range: 10 12
New Value2: 254.95
Range: 10 11
New Value1: 254.71
Range: 12 12
New Value2: 265.66
Range: 10 10
Last Value: 270.19
Range: 11 11
Last Value: 270.19
```

```
$ cat how_to_triage_cpu2017_file_for_perf.sh
#!/bin/sh
#set -x
if [ $# != 5 ]
then
        echo "ERROR: how_to_triage_cpu2017_file_for_perf.sh takes exactly 5 argument"
        echo "Usage: how_to_triage_cpu2017_file_for_perf.sh <benchnum> <specconfig> <comp> <baseopt>  <optimization switch>"
        echo "Usage: how_to_triage_cpu2017_file_for_perf.sh <538> <spec2017.cfg> /local/foo/comp/gcc-9.0.0 <-march=native -mfpmath=sse -foption> <-foption=none>"
        exit
fi
export benchnum=${1}
export cfg=${2}
export comp=${3}
export baseopt="${4}"
export optswitch="${5}"
export GCC_DIR=${comp}
export BASEOPT="${baseopt}"
export corecount=`lscpu | grep '^CPU(s):' | cut -f2 -d':' | sed -e 's/ //g'`
export bits="64"
cd /local/foo/cpu2017
case $benchnum in
500)
        export benchname="perlbench_r"
        export runtype="refrate"
        ;;
600)
        export benchname="perlbench_s"
        export runtype="refspeed"
        ;;
502)
        export benchname="gcc_r"
        export runtype="refrate"
        ;;
602)
        export benchname="gcc_s"
        export runtype="refspeed"
        ;;
503)
        export benchname="bwaves_r"
        export runtype="refrate"
        ;;
603)
        export benchname="bwaves_s"
        export runtype="refspeed"
        ;;
505)
        export benchname="mcf_r"
        export runtype="refrate"
        ;;
605)
        export benchname="mcf_s"
        export runtype="refspeed"
        ;;
507)
        export benchname="cactuBSSN_r"
        export runtype="refrate"
        ;;
607)
        export benchname="cactuBSSN_s"
        export runtype="refspeed"
        ;;
508)
        export benchname="namd_r"
        export runtype="refrate"
        ;;
510)
        export benchname="parest_r"
        export runtype="refrate"
        ;;
511)
        export benchname="povray_r"
        export runtype="refrate"
        ;;
519)
        export benchname="lbm_r"
        export runtype="refrate"
        ;;
619)
        export benchname="lbm_s"
        export runtype="refspeed"
        ;;
520)
        export benchname="omnetpp_r"
        export runtype="refrate"
        ;;
620)
        export benchname="omnetpp_s"
        export runtype="refspeed"
        ;;
521)
        export benchname="wrf_r"
        export runtype="refrate"
        ;;
621)
        export benchname="wrf_s"
        export runtype="refspeed"
        ;;
523)
        export benchname="xalancbmk_r"
        export runtype="refrate"
        ;;
623)
        export benchname="xalancbmk_s"
        export runtype="refspeed"
        ;;
525)
        export benchname="x264_r"
        export runtype="refrate"
        ;;
625)
        export benchname="x264_s"
        export runtype="refspeed"
        ;;
526)
        export benchname="blender_r"
        export runtype="refrate"
        ;;
527)
        export benchname="cam4_r"
        export runtype="refrate"
        ;;
627)
        export benchname="cam4_s"
        export runtype="refspeed"
        ;;
628)
        export benchname="pop2_s"
        export runtype="refspeed"
        ;;
531)
        export benchname="deepsjeng_r"
        export runtype="refrate"
        ;;
631)
        export benchname="deepsjeng_s"
        export runtype="refspeed"
        ;;
538)
        export benchname="imagick_r"
        export runtype="refrate"
        ;;
638)
        export benchname="imagick_s"
        export runtype="refspeed"
        ;;
541)
        export benchname="leela_r"
        export runtype="refrate"
        ;;
641)
        export benchname="leela_s"
        export runtype="refspeed"
        ;;
544)
        export benchname="nab_r"
        export runtype="refrate"
        ;;
644)
        export benchname="nab_s"
        export runtype="refspeed"
        ;;
548)
        export benchname="exchange2_r"
        export runtype="refrate"
        ;;
648)
        export benchname="exchange2_s"
        export runtype="refspeed"
        ;;
549)
        export benchname="fotonik3d_r"
        export runtype="refrate"
        ;;
649)
        export benchname="fotonik3d_s"
        export runtype="refspeed"
        ;;
554)
        export benchname="roms_r"
        export runtype="refrate"
        ;;
654)
        export benchname="roms_s"
        export runtype="refspeed"
        ;;
557)
        export benchname="xz_r"
        export runtype="refrate"
        ;;
657)
        export benchname="xz_s"
        export runtype="refspeed"
        ;;
*)
        echo "Bench $benchnum not supported"
        exit
esac


export rangeb=1
export label=`date +%Y%m%d$$`
. ./shrc
ulimit -s unlimited
rm -rf benchspec/CPU/*/{build,run,exe}/*
runcpu --fake --loose -a run --tune base --label ${label} --config ${cfg} ${benchnum} > /dev/null 2>&1
#exit
go ${benchnum} > /dev/null 2>&1
cd build/build_base_${label}.0000
if [ $benchnum -eq 511 ] || [ $benchnum -eq 521 ] || [ $benchnum -eq 525 ] || [ $benchnum -eq 526 ] || [ $benchnum -eq 527 ] || [ $benchnum -eq 538 ] || [ $benchnum -eq 621 ] || [ $benchnum -eq 625 ] || [ $benchnum -eq 627 ] || [ $benchnum -eq 638 ]
then
        specmake clean TARGET=${benchname} > /dev/null 2>&1
        specmake TARGET=${benchname} > ${benchname}.cmd 2> ${benchname}.cmd.error
else
        specmake clean > /dev/null 2>&1
        specmake > ${benchname}.cmd 2> ${benchname}.cmd.error
fi
export benchname_exe=`tail -1 ${benchname}.cmd | sed -e "s/^.*-o //g"`
export rangee=`wc -l ${benchname}.cmd | cut -f1 -d' '`
let "rangee=($rangee-1)"
cp ${benchname_exe} ../../run/run_base_${runtype}_${label}.0000/${benchname_exe}_base.${label}
go ${benchname} run > /dev/null 2>&1
cd run_base_${runtype}_${label}.0000
/usr/bin/time -f "%e" specinvoke > /tmp/ll 2>&1
export old_value=`grep -v exit /tmp/ll`
echo "Intital value: $old_value"
dorun()
{
        go ${benchnum} > /dev/null 2>&1
        cd build/build_base_${label}.0000
        sed -e "$1,$2s/$/ ${optswitch}/g" ${benchname}.cmd > ${benchname}.cmd_$1_$2
        chmod +x ${benchname}.cmd_$1_$2
        ./${benchname}.cmd_$1_$2
        cp ${benchname_exe} ../../run/run_base_${runtype}_${label}.0000/${benchname_exe}_base.${label}
        go ${benchname} run > /dev/null 2>&1
        cd run_base_${runtype}_${label}.0000
        /usr/bin/time -f "%e" specinvoke > /tmp/$$ 2>&1
}

while(true)
do
        #create new range.
        rangem=(rangeb+rangee)/2
        let "rangem=($rangeb+$rangee)/2"
        if [[ "$rangeb+1" -eq "$rangee" ]]
        then
                #echo "RangeB: $rangeb"
                #echo "RangeE: $rangee"
                dorun $rangeb $rangeb
                export last_value=`grep -v exit /tmp/ll`
                echo "Range: $rangeb $rangeb"
                echo "Last Value: $last_value"
                let "rangeb=$rangeb+1"
                dorun $rangeb $rangeb
                export last_value=`grep -v exit /tmp/ll`
                echo "Range: $rangeb $rangeb"
                echo "Last Value: $last_value"
                exit
        elif [[ "$rangeb" -eq "$rangee" ]]
        then
                #echo "RangeB: $rangeb"
                #echo "RangeE: $rangee"
                dorun $rangeb $rangee
                export last_value=`grep -v exit /tmp/ll`
                echo "Range: $rangeb $rangee"
                echo "Last Value: $last_value"
                exit
        fi
        let "rangemn=$rangem+1"
        #calculate new_value1
        dorun $rangeb $rangem
        export new_value1=`grep -v exit /tmp/$$`
        echo "Range: $rangeb $rangem"
        echo "New Value1: $new_value1"
        dorun $rangemn $rangee
        export new_value2=`grep -v exit /tmp/$$`
        echo "Range: $rangemn $rangee"
        echo "New Value2: $new_value2"
        if (( $(echo "$new_value1 < $new_value2" | bc -l) ))
        then
                if (( $(echo "$new_value1 < $old_value" | bc -l) ))
                then
                        export rangee=$rangem
                        export old_value="`echo $new_value1+\(\($new_value1/100\)*1\) | bc -l`"
                else
                        echo "Search failed for ${benchnum}"
                        exit
                fi
        else
                if (( $(echo "$new_value2 < $old_value" | bc -l) ))
                then
                        export rangeb=$rangemn
                        export old_value="`echo $new_value2+\(\($new_value2/100\)*1\) | bc -l`"
                else
                        echo "Search failed ${benchnum}"
                        exit
                fi

        fi
done
```

```
$ cat config/spec2017.cfg
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
