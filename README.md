# Running PyPSA-Eur in GenomeDK


Description
-----------
The instructions below describe the steps to follow to successfully install and build [PyPSA-Eur](https://pypsa-eur.readthedocs.io) version 0.8.0 or higher in [GenomeDK](https://genome.au.dk), a high-performance computing facility for the life sciences in Denmark serving scientists, students and SMEs. Although GenomeDK was primarily meant for the life sciences, this facility is equally suitable to run any CPU-intensive task including energy system models.


Steps
-----

1. Request an account to get access to GenomeDK [here](https://console.genome.au.dk/user-requests/create).

2. Once the account has been created (an email will be sent to the user announcing this fact), log in GenomeDK through [ssh](https://en.wikipedia.org/wiki/Secure_Shell) by opening a terminal and executing the following (replace `<username>` with the information provided when requesting an account in step `1`):

```Bash
ssh <username>@login.genome.au.dk
```

3. Once logged in GenomeDK, users are initially assigned to one of its front-end machines. Given that GenomeDK front-end machines are configured with a maximum limit of *1000* processes a user is allowed to launch at any point in time (which is a rather low limit), PyPSA-Eur will likely fail to build showing messages such as `ERROR; return code from pthread_create() is 11`, `OpenBLAS blas_thread_init: pthread_create failed for thread 9 of 16` and `Resource temporarily unavailableOpenBLAS blas_thread_init: RLIMIT_NPROC 1000 current, 1000 max`. The solution to this is to build PyPSA-Eur in one of GenomeDK back-end machines, where this limit is much higher (e.g. *1544749*). Consequently, from a GenomeDK front-end machine, log in to one of its back-end machines by executing the following:

```Bash
srun --mem 16G --pty bash
```

   After some seconds, a back-end machine with 16 GB of RAM memory and configured with the [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) shell will be assigned to the user. It is important to explicitly allocated (at least) this amount of RAM memory to the back-end machine given that the default value (set to 10 GB by GenomeDK administrators currently) will likely make PyPSA-Eur failing to build as some of its building rules require a lot of RAM memory, forcing the OS to abruptly terminate the process and show the cryptic message `Killed`.

4. Once logged in one of GenomeDK back-end machine, open a directory named `workspace` by executing:

```Bash
mkdir workspace
```

5. Go in this directory by executing:

```Bash
cd workspace
```

6. Download Anaconda installation file for Linux 64 bit [here](https://www.anaconda.com/download). Typically, an Anaconda installation file for Linux 64 bit follows a naming convention similar to `Anaconda3-<version>-Linux-x86_64.sh`, where `<version>` refers to a specific version of Anaconda (e.g. `Anaconda3-2023.03-1-Linux-x86_64.sh`).

7. Assuming that the Anaconda installation file is stored in directory `workspace`, execute the following to install it:

```Bash
bash Anaconda3-<version>-Linux-x86_64.sh
```

8. Download Gurobi from [here](https://www.gurobi.com/downloads/gurobi-software) and install it in directory `workspace` (in GenomeDK).

9. To run Gurobi, a license is needed. The Aarhus University runs a server which contains licenses for disparate software packages/tools/systems used in this institution. To have the installed Gurobi consuming the license stored in this server, create a file called, e.g., `gurobi.lic` (in directory `workspace`) containing the following two lines:

```
TOKENSERVER=licsrv01.uni.au.dk
PORT=41954
```

10. Add to the `.bashrc` file (found in the home root of the user) the following four lines (replace `<username>` with the information provided when requesting an account in step `1` and `<version>` with the version of Gurobi installed in step `8`):

```
   export GRB_LICENSE_FILE="/home/<username>/workspace/gurobi.lic"
   export GUROBI_HOME="/home/<username>/workspace/gurobi-<version>/linux64"
   export PATH=:${GUROBI_HOME}/bin:$PATH
   export LD_LIBRARY_PATH=${GUROBI_HOME}/lib
```

11. Install Gurobi Python package in Anaconda by executing (replace `<version>` with the version of Gurobi installed in step `8`):

```Bash
conda install -c gurobi gurobi-<version>
```

12. After installing Gurobi and enabling Anaconda to properly interact with it, clone PyPSA-Eur project by executing:

```Bash
git clone https://github.com/PyPSA/pypsa-eur.git
```

13. Go in directory `pypsa-eur` by executing:

```Bash
cd pypsa-eur
```

14. Create an Anaconda environment tailored for PyPSA-Eur by doing the following (this might take a while to complete):

```Bash
conda env create -f envs/environment.yaml
```

15. Activate the created Anaconda environment by executing:

```Bash
conda activate pypsa-eur
```

16. Finally, to build PyPSA-Eur execute the following (this might take a while to complete):

```Bash
snakemake -j1
```

