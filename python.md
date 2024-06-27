# Python

We will be using Jupyter notebooks for first few classes. Jupyter notebook allows us to interactively use Python. First you should use following command in your *home* directory. 
```sh
module load anaconda3/2024.2
conda create --name myenv <package-1> <package-2> ... <package-N>
```

## Using it in jobs
If you are already familiar with Python or interested in further exploring it use the following [link](https://researchcomputing.princeton.edu/support/knowledge-base/python) for instructions on using Python on Princeton clusters. This section will break it down to an easy extent.

We will be referencing [this](https://github.com/PrincetonUniversity/hpc_beginning_workshop/tree/main/python/cpu) link for the tutorial underneath.

To get started with python, we need a conda environment, something that helps us manage libraries and packages.

```sh
ssh <YourNetID>@adroit.princeton.edu
# Della users: ssh <YourNetID>@della.princeton.edu

# cd to scratch/network/<YourNetID> since we are testing and these files are not important
cd /scratch/network/<YourNetID>/python_test
# Della users:  cd /scratch/gpfs/<YourNetID>/python_test

# Load conda module to be able to access conda executable and environment
module load anaconda3/2024.2
# Create the environment
conda create --name ml-env scikit-learn pandas matplotlib --channel conda-forge
# Activate the environment
conda activate ml-env
```

Next, create a python file called `matrix-inverse.py`
```sh
touch matrix-inverse.py && nano matrix-inverse.py

import numpy as np
N = 3
X = np.random.randn(N, N)
print("X =\n", X)
print("Inverse(X) =\n", np.linalg.inv(X))
```

Below is the `job.slurm` file that should go in the same directory:
```sh
#!/bin/bash
#SBATCH --job-name=matinv        # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=1               # total number of tasks across all nodes
#SBATCH --cpus-per-task=1        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G is default)
#SBATCH --time=00:01:00          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=begin        # send email when job begins
#SBATCH --mail-type=end          # send email when job ends
#SBATCH --mail-type=fail         # send email if job fails
#SBATCH --mail-user=<YourNetID>@princeton.edu

module purge
module load anaconda3/2024.2

python matrix_inverse.py
```

Lastly, submit the job then wait a few minutes to view the log.

```sh
sbatch job.slurm
```

```sh
X =
 [[ 1.34903552  0.67843791 -0.43574188]
 [ 1.60554068  0.66067333 -0.38373032]
 [-1.54970008  1.28489254  0.14557173]]
Inverse(X) =
 [[ -1.93014233   2.15752707  -0.09023234]
 [ -1.18235524   1.56870044   0.59596897]
 [-10.11145687   9.12202091   0.64855209]]
```

## Embarrassingly Parallel
This is a way to parallelize your jobs. Let's say that you have different inputs files labelled 1-1000. One way is to submit all these inputs and one script to read them all to one job. Or, you can submit one job that will distribute these inputs across different sub-jobs that will be able to parallelize and speed up calculations massively.

Please refer to [this](https://github.com/PrincetonUniversity/hpc_beginning_workshop/tree/main/job_array/python) link to learn more.


## Running Jupyter on the web browser
We will use Jupyter notebook to access the above conda environment. Please follow the research computing [instructions](https://researchcomputing.princeton.edu/support/knowledge-base/jupyter#ondemand) to run Jupyter on your web-browser on Adroit. 

First, you will need to access the on-demand websites. 
- [Della](https://mydella.princeton.edu/)
- [Adriot](https://myadroit.princeton.edu/)

Next, login and click on `My interactive sessions 📒` on menu bar.

On the sidebar, click on Jupyter.

Then, an option/form to launch a jupyter environment will pop up in front of the screen.

Select `Use JupyterLab instead of Jupyter Notebook?`.

Select `Node Type` as `skylake`

For `Anaconda3 version used for starting up jupyter interface`, select your custom python environment with all the packages installed. In our case, we would want to select the `ase` environment created underneath the `ase` section in [tools.md](tools.md) to be able to run the `ase` python package in jupyter.


