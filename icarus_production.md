## Running a campaign:
##### Example:
```shell
With icaruscode v10_06_00_01 available, we are now ready to start the data production testing. To get started, I'd ask that we start by processing the files in the sam definition gputnam_2025AProductionTest_9384_BNBMajroity through the following workflow:
stage0_run2_wcdnn_icarus.fcl
stage1_run2_icarus.fcl
cafmakerjob_icarus_data.fcl
```

## icaruspro
```shell
# login
ssh faabdalr@icarusprodgpvm01.fnal.gov

# list definition files example
samweb -e icarus list-definition-files gputnam_2025AProductionTest_9384_BNBMajroity

```

## Delete the files
After running a campaign, delete the files that are saved in ``/pnfs/sbn/data`` using samweb:
```shell
# set up the following dependencies via Spack
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


