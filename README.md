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
## THIS NEEDS TO BE UPDATED WHEN FLEX_EXTRACT WORKS

1. Create a dedicated folder in your /data/ directory
2. 

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
