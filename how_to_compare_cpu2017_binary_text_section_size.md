### How to compare cpu2017 binary size

$ ./how_to_compare_cpu2017_binary_text_section_size.sh benchspec.exp /local/skpandey/cpu2017/benchspec.ref

500.perlbench_r (0.30%)

502.gcc_r (1.03%)

503.bwaves_r (0.15%)

505.mcf_r (0.46%)

507.cactuBSSN_r (0.42%)

508.namd_r (0.08%)

510.parest_r (1.53%)

511.povray_r (0.54%)

519.lbm_r (0.78%)

520.omnetpp_r (1.76%)

521.wrf_r (0.19%)

523.xalancbmk_r (1.73%)

525.x264_r (0.25%)

526.blender_r (1.40%)

527.cam4_r (0.03%)

531.deepsjeng_r (0.48%)

538.imagick_r (0.34%)

541.leela_r (1.01%)

544.nab_r (0.40%)

548.exchange2_r (0.01%)

549.fotonik3d_r (0.05%)

554.roms_r (0.08%)

557.xz_r (0.60%)

997.specrand_fr (1.26%)

999.specrand_ir (1.26%)


```
$ cat how_to_compare_cpu2017_binary_text_section_size.sh
#!/bin/sh
if [ $# != 2 ]
then
        echo "how_to_compare_cpu2017_binary_text_section_size.sh {exp} {ref}"
        echo "Example: how_to_compare_cpu2017_binary_text_section_size.sh benchspec.exp benchspec.ref"
        exit
fi
export expresultfile="$1";
export refresultfile="$2";

export exptestnumber=(`size ${expresultfile}/CPU/*/exe/* | grep -v validate | grep -v diff | grep -v ldecod | sed -e 's/\s\s*/ /g' | sed -e 's/^\s*//g' | cut -f1 -d' ' | tail -n +2`)
export exptestname=(`size ${expresultfile}/CPU/*/exe/* | grep -v validate | grep -v diff | grep -v ldecod | sed -e 's/\s\s*/ /g' | sed -e 's/^\s*//g' | cut -f6 -d' ' | tail -n +2 | sed -e 's/^.*\/CPU\///g' | sed -e 's/\/.*//g'`)

export reftestnumber=(`size ${refresultfile}/CPU/*/exe/* | grep -v validate | grep -v diff | grep -v ldecod | sed -e 's/\s\s*/ /g' | sed -e 's/^\s*//g' | cut -f1 -d' ' | tail -n +2`)
export reftestname=(`size ${refresultfile}/CPU/*/exe/* | grep -v validate | grep -v diff | grep -v ldecod | sed -e 's/\s\s*/ /g' | sed -e 's/^\s*//g' | cut -f6 -d' ' | tail -n +2 | sed -e 's/^.*\/CPU\///g' | sed -e 's/\/.*//g'`)

i=0
while (( $i < ${#exptestnumber[@]} ))
do
        if [ ${exptestname[$i]} = ${reftestname[$i]} ]
        then
                export per=`echo \(\(\(${exptestnumber[$i]}-${reftestnumber[$i]}\)/${reftestnumber[$i]}\)*100\) | bc -l`
                #echo ":$per:"
                #echo :${exptestname[$i]}:
                printf "%s (%.2f%s)\n" ${exptestname[$i]} $per '%'
        fi
        i=$((i+1))
done
