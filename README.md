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


1. Get the flex_extract source code, make sure to get the dev branch
Get the flex_extract dev branch for use with FLEXPART 10.4 following the advice in: https://www.flexpart.eu/ticket/297

```bash
git clone --single-branch --branch dev https://www.flexpart.eu/gitmob/flex_extract
```

2. Load some modules to ensure we can compile and later run the script extract_data.sh (see step 7)
```bash
module load gcc/9.4.0
module load eccodes/2.21.0-serial
```
In your python environment, ensure you install Genshi
```bash
module load pangeo-meso/2023.04.15
pip install Genshi
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

Setup your CDS api key as instructed by creating this file with your key information
```bash
$HOME/.cdsapirc
```

Install the CDS API in your pangeo environment
```bash
pip install cdsapi
```

7. Go to the Run folder for flex_extract
For, exmaple
```bash
cd ~/FLEXPART/FLEXPART/flex_extract_dev/Run
```
Create a script to use the queue to extract data called extract_data.sh
```bash
#!/bin/bash
#SBATCH --job-name=Run_FP           # nom du job
#SBATCH --partition=zen16                   # Nom d'une partition pour une exÃ©tion cpu
#SBATCH --ntasks=1                          # nombre de taches
#SBATCH --ntasks-per-node=1                 # nombre de taches MPI par noeud
#SBATCH --mem=10GB                          # memory limit
#SBATCH --time=04:00:00                     # temps d execution maximum demande (HH:MM:SS)
#SBATCH --output=Flex_extract_%j.out       # nom du fichier de sortie
#SBATCH --error=Flex_extract_%j.out        # nom du fichier d'erreur (ici en commun avec la sortie)


module purge
module load netcdf-fortran/4.5.3-serial
module load jasper/2.0.32
module load eccodes/2.21.0-serial
module load pangeo-meso/2023.04.15
~                                 
./run_local.sh
```

8. Modify run_local.sh to be for public ERA5 data, for the dates you need data, with the input and ouput directories (for example):
```bash
# AVAILABLE COMMANDLINE ARGUMENTS TO SET
# 
# THE USER HAS TO SPECIFY THESE PARAMETERS:

QUEUE=None
START_DATE=20180809
END_DATE=20180809
DATE_CHUNK=None
JOB_CHUNK=3
BASETIME=None
STEP=None
LEVELIST=None
AREA=None
INPUTDIR='/data/thomas/FLEXPART-In/ERA5-dev'
OUTPUTDIR='/data/thomas/FLEXPART-In/ERA5-dev'
PP_ID=None
JOB_TEMPLATE=None
CONTROLFILE='CONTROL_EA5.global'
DEBUG=0
REQUEST=2
PUBLIC=1
```

9. Launch flex_extract via the queue using
```bash
sbatch extract_data.sh
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
