## Running a campaign:
#### Example:
With icaruscode ``v10_06_00_01`` available, we are now ready to start the data production testing. To get started,
I'd ask that we start by processing the files in the sam definition ``gputnam_2025AProductionTest_9435_BNBMajroity``
through the following workflow:
``stage0_run2_wcdnn_icarus.fcl``
``stage1_run2_icarus.fcl``
``cafmakerjob_icarus_data.fcl``

#### Steps:
Clone a similar campaign then go to GUI editor:
   
##### Update:
```shell
 name: yearA_experiment_RunNo_version_something-intuitive  , e.g. 2025A_ICARUS__BNB_Run9435_v10_VAL
 software_version: v10_06_00_01
```

##### Campaign Stages:
   
###### Stage0:
```shell
name: stage0_onbeam
dataset: gputnam_2025AProductionTest_9435_BNBMajroity
```
   
###### param_override:
```shell
stage: stage0_onbeam
streamname: bnbmajority
campaign: 2025_ICARUS__BNB_Run9435_v10_VAL
fclfile: stage0_run2_wcdnn_icarus.fcl
Ojob_output.addoutpu: compressed_data*stage0.root
Ojob_output.dest: /pnfs/%(experiment)s/scratch/users/%(experiment)spro/dropbox/data/poms_production
                  /%(experiment)s_Icaruspro_2025_wcdnn/% (version)s/bnbmajority/stage0_9435/
general_dest:     /pnfs/%(experiment)s/scratch/users/%(experiment)spro/dropbox/data/poms_%(prodstatus)s
                 /%(experiment)s_%(sample)s/%(version)s/% (streamname)s/stage0/
sample: Icaruspro_2025_wcdnn
prodtype: gputnam_9435
```

###### Stage1:
```shell
name: POT_stage1_caf_larcv_stage1onDisk
dataset: gputnam_2025AProductionTest_9435_BNBMajroity
```
   
###### param_override:
```shell
stage: POT_stage1_caf_larcv_stage1onDisk
streamname: bnbmajority
streaminfo: bnb
fclfile: run_bnbinfo_sbn.fcl
fclfile_1: stage1_run2_icarus.fcl
fclfile_2: cafmakerjob_icarus_data.fcl
Oexecutable_3.name: true
Ojob_output.dest: /pnfs/icarus/scratch/users/icaruspro/dropbox/data/poms_production/icarus_Icaruspro_2025_Run2_wcdnn/% 
                       (version)s/bnbmajority/stage1_9435/
sample: Icaruspro_2025_wcdnn
prodtype: gputnam_9435
```

#### Submit a test:
```shell
split_type: limitn --> 20
```
   
#### Path to the config file:   
```shell
/exp/icarus/app/poms_test/pr_cfg_numi/run2_rep_POT_promita.cfg
```

## icaruspro
```shell
# login
ssh faabdalr@icarusprodgpvm01.fnal.gov

# list definition files example
samweb -e icarus list-definition-files gputnam_2025AProductionTest_9384_BNBMajroity

```

## Delete the files
After running a campaign, delete the files that are saved in ``/pnfs/sbn/data/sbn_fd`` using samweb:
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


