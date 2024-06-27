# Simulation Tools

## Atomistic simulation environment (ASE)

[ASE](https://wiki.fysik.dtu.dk/ase/index.html) is a Python based simulation environment that can be used to manage/create/control your simulations on 
multiple standard simulation softwares. We are going to cover some use cases of ASE. We will use it through the interactive Python environment.  

```{admonition} Create an ASE environment on Adroit using conda
Execute the following commands
 

```sh
#Load the anaconda module
module load anaconda3/2024.2

#Create a conda environment ASE
conda create --name ase python=3.8 ase pandas matplotlib --channel conda-forge

#To check the environment execute the following
conda activate ase

#You should see a small **(ase)** text at the start of your command line

conda install ase-notebook --channel conda-forge

```

## LAMMPS

### Typical input files required (The names can be changed)

- in.lammps: Input file. This file has all the parameters controlling the simulations (e.g. Thermostat, Barostate, Ensemble) 
- geom.xyz: (optional) Geometry file. This file describes the coordinates of your simulation system. Alternatively, you can also define your system geometry in the input file.
- potential file: (optional) A file defining the MD potential that you are using.

### LAMMPS example script

Installing LAMMPS

```sh
#!bin/bash

VERSION=17Apr2024
wget https://github.com/lammps/lammps/archive/refs/tags/patch_${VERSION}.tar.gz
tar zvxf patch_${VERSION}.tar.gz
cd lammps-patch_${VERSION}
mkdir build && cd build

# include the modules below in your Slurm scipt
module purge
module load intel/19.1.1.217 intel-mpi/intel/2019.7
```

#### Making with cmake

**Adriot**

```sh
cmake3 \
-D CMAKE_INSTALL_PREFIX=$HOME/.local \
-D LAMMPS_MACHINE=adroit \ 
-D CMAKE_BUILD_TYPE=Release \
-D CMAKE_CXX_COMPILER=icpc \
-D CMAKE_CXX_FLAGS_RELEASE="-Ofast -xHost -DNDEBUG" \
-D CMAKE_Fortran_COMPILER=/opt/intel/compilers_and_libraries_2020.1.217/linux/bin/intel64/ifort \
-D BUILD_OMP=yes \
-D BUILD_MPI=yes \
-D PKG_KSPACE=yes -D FFT=MKL -D FFT_SINGLE=no \
-D PKG_OPENMP=yes \
-D PKG_MOLECULE=yes \
-D PKG_RIGID=yes \
-D PKG_REAXFF=yes\
-D ENABLE_TESTING=yes ../cmake
```

**Della**

```sh
cmake3 \
-D CMAKE_INSTALL_PREFIX=$HOME/.local \
-D LAMMPS_MACHINE=della \
-D CMAKE_BUILD_TYPE=Release \
-D CMAKE_CXX_COMPILER=icpc \
-D CMAKE_CXX_FLAGS_RELEASE="-Ofast -xHost -DNDEBUG" \
-D CMAKE_Fortran_COMPILER=/opt/intel/compilers_and_libraries_2020.1.217/linux/bin/intel64/ifort \
-D BUILD_OMP=yes \
-D BUILD_MPI=yes \
-D PKG_KSPACE=yes -D FFT=MKL -D FFT_SINGLE=no \
-D PKG_OPENMP=yes \
-D PKG_MOLECULE=yes \
-D PKG_RIGID=yes \
-D PKG_REAXFF=yes\
-D ENABLE_TESTING=yes ../cmake
```

**Finish with compiling the executable for lammps**

```sh
make -j 10
#make test
make install
```


Once this has been compiled, execute `ls` and you will find a file named `lmps_adriot` or `lmps_della` in the same directory.

Move that file into a folder in which you store your executables. Normally, I would recommend storing it in `/home/<NET-ID>/.local/bin`, since it is already in `$PATH`.

However, we will try to use a LAMMPS executable that has been already compiled that can be found on canvas.

Use the following job submission script to submit a LAMMPS job
```sh
#!/bin/bash
#SBATCH --job-name=lj-melt       # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=4               # total number of tasks across all nodes
#SBATCH --cpus-per-task=1        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=1G         # memory per cpu-core (4G is default)
#SBATCH --time=00:05:00          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=begin        # send email when job begins
#SBATCH --mail-type=end          # send email when job ends
#SBATCH --mail-user=<YourNetID>@princeton.edu

module purge
module load intel/19.1.1.217 intel-mpi/intel/2019.7
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Argument as such: srun [executable location] [mode (-in)] [input file (.melt, .lammps)]
# Change the executable location to where you store your compiled lammps file.
srun /home/al9001/.local/bin/lmp_adroit -in in.melt
```

The input file can be copied from below

```sh
units           lj
atom_style      atomic

lattice         fcc 0.8442
region          box block 0 30 0 30 0 30
create_box      1 box
create_atoms    1 box
mass            1 1.0

velocity        all create 1.0 87287

pair_style      lj/cut 2.5
pair_coeff      1 1 1.0 1.0 2.5

neighbor        0.3 bin
neigh_modify    every 20 delay 0 check no

fix             1 all nve
fix             2 all langevin 1.0 1.0 1.0 48279

timestep        0.005

thermo          5000
run             10000
```


## OVITO

[OVITO](https://www.ovito.org/) is an easy to use visualization software that can also be used to convert different geometry file formats. You can download and install the free version locally on your computer for visualization.

## PACKMOL

[PACKMOL](https://m3g.github.io/packmol/download.shtml) is a tool to generate initial MD configurations for molecules. Please download it
using the link with wget in your home folder (/home/Princeton_ID/). 

Steps to find download link:
1. Click on given link
2. Click on "[LATEST RELEASE]". This will bring you to a page on Github.
3. Over the hyperlink with the text `.tar.gz`, right click and select `copy link`.

I will download the most recent version of packmol with this command with the url from the steps above.
```
wget https://github.com/m3g/packmol/archive/refs/tags/v20.14.4+docs1.tar.gz
```

The download file should be renamed into "packmol.tar.gz"

Unzip this file using the following command

```sh
tar -xf packmol.tar.gz
``` 
Change the directory to Packmol and compile it using the command

```sh
make
```
This would produce a file called `packmol`, you can then move this into your `~/.local/bin` folder.

Also download the [examples.zip](https://m3g.github.io/packmol/examples/examples.zip) file from the website and upload it to your home folder (or, use wget). Unzip it using the following command

```sh
unzip examples.zip
``` 

Change the directory to examples folder and execute following command to generate an example geometry.

```sh
# Remember that it is "<" not ">". If you use ">" in your command, your input file will be wiped clean and you will have to re-enter your inputs.
/location_to_packmol/packmol < mixture.inp
```


## VASP

### Some basics 

Files used to run VASP calculations
- INCAR: This file describes the input parameters required for different calculations (For example: A molecule will have different parameters than a crystal)
- KPOINTS: Description of reciprocal space
- POSCAR: Define the geometry of your system
- POTCAR: Concatenated pseudopotentials for elements of your system 

### Running VASP on Adroit

We will try to use a VASP executable that has been already compiled

Use the following job submission script to submit a VASP job
```sh
#!/bin/bash
#SBATCH --job-name=vasp          # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks-per-node=8      # total number of tasks per node
#SBATCH --cpus-per-task=2        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G is default)
#SBATCH --time=00:01:00          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=begin        # send email when job begins
#SBATCH --mail-type=end          # send email when job ends
#SBATCH --mail-user=<YourNetID>@princeton.edu

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export PATH=$PATH:/home/al9001/software/vasp.6.2.1/bin/

module purge
module load intel/2021.1.2 intel-mpi/intel/2021.1.1

srun vasp_std
```

