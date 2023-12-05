# Download and compile FLEXPART v10.4 on spirit (serial mode)

1. Clone this repository and move into the directory.

2. Run the command `make sources`. This will prepare the FLEXPART sources,
   mainly:

   - Download the sources from the official FLEXPART website.

   - Check the integrity of the downloaded file.

   - Untar and patch the sources. In particular, it patches the FLEXPART
     makefile to make it usable on the [spirit
     supercalculator](https://mesocentre.ipsl.fr/).

3. Run the command `make flexpart-serial`. This will compile FLEXPART in serial
   mode. The resulting binary is `flexpart_v10.4/src/FLEXPART`.
