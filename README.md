# Download and compile FLEXPART v10.4 on spirit (serial mode)

1. Clone this repository and move into the directory.

2. Run the command `make sources`. This will prepare the FLEXPART sources,
   mainly:

   - Download the sources from the official FLEXPART website.

   - Check the integrity of the downloaded file.

   - Untar and patch the sources. In particular, it patches the FLEXPART
     makefile to make it usable on the [spirit
     cluster](https://mesocentre.ipsl.fr/).

3. Run the command `make flexpart-serial`. This will compile FLEXPART in serial
   mode. The resulting binary is `flexpart_v10.4/src/FLEXPART`.


# Prepare the meteorological data for running FLEXPART
The following problems with flex_extract have been noted due to the fact that EMOSLIB is now retired.  Documented here: https://www.flexpart.eu/ticket/312


1. Get the flex_extract source code
Get the flex_extract dev branch for use with FLEXPART 10.4 following the advice in: https://www.flexpart.eu/ticket/297

```bash
git clone --single-branch --branch dev https://www.flexpart.eu/gitmob/flex_extract
```

2. Load some modules to ensure we can compile
```bash
module load gcc/9.4.0
module load eccodes/2.21.0-serial
```

3. Ensure you are working in the folder where you have downloaded flex_extract, for example:
```bash
cd /home/thomas/FLEXPART/FLEXPART/flex_extract_dev
```

4. Modify the setup_local.sh file, these options for an instalation in the folder where you are working
```bash
# THE USER HAS TO SPECIFY THESE PARAMETERS
#
export ECCODES_ROOT=/net/nfs/tools/u20/22.3/PrgEnv/gcc/linux-ubuntu20.04-zen2/eccodes/2.21.0-gcc-9.4.0-rdneibdekiizqpmifsvn3t3qymvsqa6k
echo $ECCODES_ROOT
TARGET='local'
MAKEFILE='makefile_local_gfortran'
ECUID=None
ECGID=None
GATEWAY=None
DESTINATION=None
INSTALLDIR=None
JOB_TEMPLATE=''
CONTROLFILE='CONTROL_EA5'
```

5. Compile and configure using
```bash
./setup_local.sh
```

6. Setup to use the Copernicus Climate Data Store on Spirit
   
Register at: https://cds.climate.copernicus.eu/

Setup the CDS API on Spirit by following these instructions: https://cds.climate.copernicus.eu/api-how-to

One way to do these steps is to operate within the pangeo-meso environment by loading this module:
```bash
module load pangeo-meso/2023.04.15
```

# Run FLEXPART v10.4 on spirit (serial mode)

### Prepare the run folder
1. Create a dedicated run directory in your $HOME or $SCRATCH
2. ln -sf the FLEXPART executable file
3. copy the options folder from the FLEXPART source code
4. copy the files mkAVAIL_v8.py, pathnames and launch_flexpart.sh from /home/lapere/test_flexpart/

### Setup the run
1. Generate the AVAILABLE FILE
python mkAVAIL_v8.py -s 2017010100 -e 2017123121 -m EI -p /data/lapere/FLEXPART/input/
where -s is the start date of the files you need, -e is the end date, -m is the meteorological input files prefix, -p is the path where these files are located
2 . Edit the pathnames file:
   - 1st line should be ./options/
   - 2nd line is the path to the run output directory (must be created before launching the run)
   - 3rd line is the path of the meteorology input file
   - 4th line is the path to the AVAILABLE file
3. Edit the options/COMMAND, options/RELEASES and options/OUTGRID files to setup the output you want

### Launch the run
sbatch launch_flexpart.sh
