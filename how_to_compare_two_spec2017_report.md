### How to compare two CPU 2017 spec perf report.

Manually calculating spec 2017 performance improvement/degradation is cumbersome and error prone.

```
$ cat how_to_compare_spec_result.sh
#!/bin/sh
if [ $# != 2 ]
then
        echo "how_to_compare_spec_result.sh {exp} {ref}"
        echo "Example: how_to_compare_spec_result.sh cpu2017/result/CPU2017.039.fprate.refrate.txt cpu2017/result/CPU2017.040.fprate.refrate.txt"
        exit
fi
export expresultfile="$1";
export refresultfile="$2";

export exptestnumber=(`grep '*\|2017_rate_base'  $expresultfile | grep -v release | sort | uniq | sed -e 's/\s*\*//g' | sed 's/[ ][ ]*/ /g' | cut -f1,4 -d' ' | sed -e 's/^ /Geomean: /g' | cut -f2 -d' '`)
export exptestname=(`grep '*\|2017_rate_base'  $expresultfile | grep -v release | sort | uniq | sed -e 's/\s*\*//g' | sed 's/[ ][ ]*/ /g' | cut -f1,4 -d' ' | sed -e 's/^ /Geomean: /g' | cut -f1 -d' '`)

export reftestnumber=(`grep '*\|2017_rate_base'  $refresultfile | grep -v release | sort | uniq | sed -e 's/\s*\*//g' | sed 's/[ ][ ]*/ /g' | cut -f1,4 -d' ' | sed -e 's/^ /Geomean: /g' | cut -f2 -d' '`)
export reftestname=(`grep '*\|2017_rate_base'  $refresultfile | grep -v release | sort | uniq | sed -e 's/\s*\*//g' | sed 's/[ ][ ]*/ /g' | cut -f1,4 -d' ' | sed -e 's/^ /Geomean: /g' | cut -f1 -d' '`)

i=0
while (( $i < ${#exptestnumber[@]} ))
do
        if [ ${exptestname[$i]} = ${reftestname[$i]} ]
        then
                export per=`echo \(\(\(${exptestnumber[$i]}-${reftestnumber[$i]}\)/${reftestnumber[$i]}\)*100\) | bc -l`
                printf "%s (%.2f%s)\n" ${exptestname[$i]} $per '%'
        fi
        i=$((i+1))
done


$ how_to_compare_spec_result.sh cpu2017/result/CPU2017.040.fprate.refrate.txt cpu2017/result/CPU2017.039.fprate.refrate.txt
503.bwaves_r (-0.58%)
507.cactuBSSN_r (-0.59%)
508.namd_r (-0.39%)
510.parest_r (-0.35%)
511.povray_r (-1.18%)
519.lbm_r (-0.48%)
521.wrf_r (-0.53%)
526.blender_r (1.38%)
527.cam4_r (-0.19%)
538.imagick_r (-5.38%)
544.nab_r (0.00%)
549.fotonik3d_r (0.88%)
554.roms_r (0.00%)
Geomean: (-0.63%)

