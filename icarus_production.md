## Delete the files
After running a production delete the files that are saved in /pnfs/sbn/data
From an ICARUS GPVM, set up the following dependencies via Spack:
 ```shell
source /cvmfs/fermilab.opensciencegrid.org/packages/common/setup-env.sh
spack load fife-utils@3.7.4 os=fe

# count the files
samweb -e icarus count-files "defname:<defname>"

# list the files
samweb -e icarus list-files "defname:<defname>"
samweb -e icarus list-files --summary "defname:<defname>"

# delete the files
sam_retire_dataset -e icarus --name <defname>

```


