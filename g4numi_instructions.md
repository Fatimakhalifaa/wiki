>[!note]
> For all steps, start on a FNAL GPVM and make sure you are inside the SL7 container. As of July 2024, we have not prepared G4NuMI as a Spack project compatible with AlmaLinux9 directly.
# Build Instructions
These are an adaptation of the instructions in the [g4numi wiki](https://cdcvs.fnal.gov/redmine/projects/numi-beam-sim/wiki/How_to_build_and_run_the_main_modern_g4_branch)
```sh
# 1. `cd` into your icarus `app` area:
cd /exp/icarus/app/users/$USER

# 2. Make directories for the intermediate build files and final compiled software to go
# this will create the following directory structure:
# g4numi/
#     - build/
#     - local_products/
mkdir -p g4numi/{build,local_products}

# 3. `cd` into the `g4numi` directory and clone the repository into a `src` directory
cd g4numi

# Clone the repository
git clone http://cdcvs.fnal.gov/projects/numi-beam-sim-g4numi ./src
```
3. Run the following commands or create a shell script with the content in `g4numi`:

```sh
# setup.sh

export PROJECT="numi_beamsim" # can be anything, just make it relevant to your current task
export EXPERIMENT=icarus
export JOBSUB_GROUP=icarus
# this one is where the output files will go
export OUTDIR=/pnfs/$EXPERIMENT/scratch/users/$USER/$PROJECT

source /cvmfs/fermilab.opensciencegrid.org/products/genie/bootstrap_genie_ups.sh

export WORKPATH=/exp/icarus/app/users/$USER/g4numi
export PRODUCTS=$PRODUCTS:$WORKPATH/local_products

# make the output directory if it doesn't already exist
[[ -d $OUTDIR ]] || mkdir -pv $OUTDIR
```
4. Source the script
```sh
source setup.sh
```
5. This next command makes it possible to set up the locally built version of g4numi via UPS.
```sh
cp -va /cvmfs/fermilab.opensciencegrid.org/products/genie/local/.upsfiles $WORKPATH/local_products
```
6. `cd` into the `src` directory and checkout the `main_modern_g4_new_target` branch
	- `main_modern` is the version of G4NuMI implementing `Geant4.10.4`
	- `new_target` incorporates the newer GDML with the updated 1 MW target:
		- Wider fins (7.4 mm --> 9.0 mm)
		- 4 additional winged fins at the upstream end of the target
	- For more information about the GDML, contact Pavel Snopok.

```sh
cd src
git checkout main_modern_g4_new_target
```

>[!warning]
>PLEASE make sure `g3Chase = true` on line 220 of `src/NumiDataInput.cc`

7. `cd` into the `build` directory and start the compilation
```sh
cd ../build

# if setup_for_development fails you may need to run:
# setup cetbuildtools v7_13_02

source ../src/ups/setup_for_development -d

setup ninja v1_10_2

buildtool -i -I ../local_products -j$(nproc) --generator ninja
```

# Running Instructions
1. Prepend the `PRODUCTS` path with the `local_products` directory where g4numi was installed, so that UPS can find it.
```sh
export PRODUCTS=$WORKPATH/local_products:$PRODUCTS
```
2. Query ups for `g4numi`
```sh
ups list -aK+ g4numi
```

Should return some output like this, but some information may differ
```
"g4numi" "dev" "Linux64bit+5.14-2.17" "debug:e19" ""
```

3. Use the info in the 2nd and 4th columns to set up the `g4numi` product:
```sh
setup g4numi dev -q debug:e19

# You can test that it worked if you invoke `which` on g4numi and verify the path is correct:

which g4numi
```


# Running the simulation on the grid
>[!warning]
> Before submitting to the grid, make sure `--OS=SL7` is removed from the job submission script, `ProcessG4NuMI_localprod.py`, and instead there is a line
> `--singularity-image /cvmfs/singularity.opensciencegrid.org/fermilab/fnal-wn-sl7:latest`


To submit grid jobs: we execute the previously mentioned python script with some arguments:

Example submission:
```sh
./ProcessG4NuMI_localprod.py --pot 500000 --outdir ${OUTDIR} --beamconfig me000z200i --n_jobs 100 --run 0
```

Good practice is to do a small test job before a full submission with parameters like `--pot 1000` and `--n_jobs 1` just to make sure things are working as expected.

Parameters:
```
--pot        <num>     -- Number of POT to simulate per job
--outdir     <path>    -- The path where you want the output files to be stored
--beamconfig <config>  -- use `me000z200i` for FHC and `me000z-200i` for RHC
--n_jobs     <num>     -- How many jobs to submit. E.g., 100 jobs with 500k POT will produce 250M total POT
--run        <num>     -- This is just a label you can assign to the output file
```

There are also a bunch of flags to generate alterations of the beam geometry

```
 Horn Options:
    --do_horn1_old_geometry
                        The new horn 1 geom (formerly known as 'alternate') is
                        now default. Use this option to use the old geometry.
                        Default = False.
    --do_horn1_fine_segmentation
                        Works for old and new horn1. Default = False.
    --horn1_position_X=HORN1_POSITION_X
                        horn 1 transverse offset (_X0). Default = 0cm.
    --horn1_position_Y=HORN1_POSITION_Y
                        horn 1 vertical offset (_Y0). Default = 0cm.
    --horn1_position_Z=HORN1_POSITION_Z
                        horn 1 longitudinal offset (_Z0). Default = 3cm.
    --horn2_position_X=HORN2_POSITION_X
                        horn 2 transverse offset (_X0). Default = 0cm.
    --horn2_position_Y=HORN2_POSITION_Y
                        horn 2 vertical offset (_Y0). Default = 0cm.
    --horn_water_mm=HORN_WATER_MM
                        Water layer on horn. Default = 1mm.

  Beam Options:
    --beam_position_X=BEAM_POSITION_X
                        beam horizontal position. Default = 0mm.
    --beam_position_Y=BEAM_POSITION_Y
                        beam vertical position. Default = 0mm.
    --beam_spotsize_X=BEAM_SPOTSIZE_X
                        beam horizontal spot size. Default = 1.4mm.
    --beam_spotsize_Y=BEAM_SPOTSIZE_Y
                        beam vertical spot size. Default = 1.4mm.

  Target Options:
    --target_position_X=TARGET_POSITION_X
                        target horizontal position. Default = 0.0cm.
    --target_position_Y=TARGET_POSITION_Y
                        target vertical position. Default = 0.0cm.
    --target_position_Z=TARGET_POSITION_Z
                        target longitudinal position. Default = -143.3cm.
```

See `./ProcessG4NuMI_localprod.py --help` for more info.
