# Running PyPSA-Eur in GenomeDK


## Description
The instructions below describe the steps to follow to successfully [install](#install) the [PyPSA-Eur](https://pypsa-eur.readthedocs.io) framework (version 0.8.0 or higher) and to [run](#run) models based on it in [GenomeDK](https://genome.au.dk), a high-performance computing facility for the life sciences in Denmark serving scientists, students and SMEs. Although GenomeDK was primarily meant for the life sciences, this facility is equally well suited to run any CPU-intensive task including energy system models.


## Install
1. Request an account to get access to GenomeDK [here](https://console.genome.au.dk/user-requests/create).

2. Once the account has been created (an email will be sent to the user announcing this fact), log in to GenomeDK through [ssh](https://en.wikipedia.org/wiki/Secure_Shell) by opening a terminal and executing the following (replace `<username>` with the information provided upon requesting an account in step `1`):

    ```bash
    privateusername@privatemachine:~$ ssh <username>@login.genome.au.dk
    ```

3. Once logged in to GenomeDK, create a directory named `workspace` by executing the following:

    ```bash
    username@genomedkfrontend:~$ mkdir workspace
    ```

4. Go in directory `workspace` by executing the following:

    ```bash
    username@genomedkfrontend:~$ cd workspace
    ```

5. Download an Anaconda installation file for Linux 64 bit by executing the following (replace `<version>` with the version of Anaconda of interest):

    ```bash
    username@genomedkfrontend:~/workspace$ wget https://repo.anaconda.com/archive/Anaconda3-<version>-Linux-x86_64.sh
    ```

6. An Anaconda installation file for Linux 64 bit is essentially a [shell script](https://en.wikipedia.org/wiki/Shell_script) file with a name similar to `Anaconda3-<version>-Linux-x86_64.sh`, where `<version>` refers to a specific version of Anaconda (e.g. `Anaconda3-2023.03-1-Linux-x86_64.sh`). Therefore, launch the Anaconda installation file by executing the following (replace `<version>` with the version of Anaconda downloaded in step `5`):

    ```bash
    username@genomedkfrontend:~/workspace$ bash Anaconda3-<version>-Linux-x86_64.sh
    ```

7. Download a Gurobi installation file for Linux 64 bit by executing the following (replace `<version>` with the version of Gurobi of interest):

    ```bash
    username@genomedkfrontend:~/workspace$ wget https://packages.gurobi.com/<version>/gurobi<version>_linux64.tar.gz
    ```

8. A Gurobi installation file for Linux 64 bit is essentially a compressed archive file with a name similar to `gurobi<version>_linux64.tar.gz`, where `<version>` refers to a specific version of Gurobi (e.g. `gurobi9.5.2_linux64.tar.gz`). Therefore, decompress the Gurobi installation file by executing the following (this will create a directory named `gurobi<version>` containing Gurobi, where `<version>` refers to the version of Gurobi downloaded in step `7`):

    ```bash
    username@genomedkfrontend:~/workspace$ tar -xzvf gurobi<version>_linux64.tar.gz
    ```

9. To be able to run Gurobi, a license is required. Fortunately, the Aarhus University maintains a server (`licsrv01.uni.au.dk`) which contains licenses for disparate software packages/tools/systems used in this institution, including Gurobi. To enable Gurobi consuming the necessary license, a file containing information on how to access the server is needed by this tool. Therefore, create such file by executing the following (this will create a file named `gurobi.lic` containing the information needed to access the server):

    ```bash
    username@genomedkfrontend:~/workspace$ echo -e "TOKENSERVER=licsrv01.uni.au.dk\nPORT=41954" > gurobi.lic
    ```

10. Add to the `.bashrc` file (found in the user home root in GenomeDK) the following four lines (replace `<username>` with the information provided upon requesting an account in step `1` and `<version>` with the version of Gurobi downloaded in step `7` without the `.` separating its major, minor and revision numbers):

    ```
    export GRB_LICENSE_FILE="/home/<username>/workspace/gurobi.lic"
    export GUROBI_HOME="/home/<username>/workspace/gurobi<version>/linux64"
    export PATH=${GUROBI_HOME}/bin:$PATH
    export LD_LIBRARY_PATH=${GUROBI_HOME}/lib
    ```

    For the addition to the `.bashrc` file to take effect on the current terminal execute `source ~/.bashrc` or, alternatively, close the terminal and open a new one (every time a new terminal is opened, the `.bashrc` file is read).

11. Install Gurobi Python package in Anaconda by executing the following (replace `<version>` with the version of Gurobi downloaded in step `7`):

    ```bash
    username@genomedkfrontend:~/workspace$ conda install -c gurobi gurobi=<version>
    ```

12. After installing Gurobi and enabling Anaconda to properly interact with it, clone PyPSA-Eur by executing the following (this will create a directory named `pypsa-eur` containing this framework):

    ```bash
    username@genomedkfrontend:~/workspace$ git clone https://github.com/PyPSA/pypsa-eur.git
    ```

13. Go in directory `pypsa-eur` by executing the following:

    ```bash
    username@genomedkfrontend:~/workspace$ cd pypsa-eur
    ```

14. Create an Anaconda environment tailored for PyPSA-Eur by executing the following (this might take a while to complete):

    ```bash
    username@genomedkfrontend:~/workspace/pypsa-eur$ conda env create -f envs/environment.yaml
    ```


## Run

1. Log in to GenomeDK through [ssh](https://en.wikipedia.org/wiki/Secure_Shell) by opening a terminal and executing the following (replace `<username>` with the information provided upon requesting an account in step `1` of the [installation](#install) procedure):

    ```bash
    privateusername@privatemachine:~$ ssh <username>@login.genome.au.dk
    ```

2. Once logged in to GenomeDK, users are initially assigned to one of its front-end machines. Given that GenomeDK front-end machines are configured with a maximum limit of *1000* processes a user may launch at any point in time (which is a rather low limit), PyPSA-Eur will likely fail to build/run showing messages such as `ERROR; return code from pthread_create() is 11`, `OpenBLAS blas_thread_init: pthread_create failed for thread 9 of 16` and `Resource temporarily unavailableOpenBLAS blas_thread_init: RLIMIT_NPROC 1000 current, 1000 max`. The solution to this is to build/run PyPSA-Eur in one of GenomeDK back-end machines, where this limit is much higher. Therefore, from a GenomeDK front-end machine, log in to one of its back-end machines by executing the following:

    ```bash
    username@genomedkfrontend:~$ srun --mem 16G --pty bash
    ```

   After some seconds, a back-end machine with 16 GB of RAM memory and configured with the [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) shell will be assigned to the user. It is crucial to explicitly allocate (at least) this amount of RAM memory to the back-end machine given that the default value (currently set to 10 GB by GenomeDK administrators) will likely make PyPSA-Eur failing to build/run as some of its ([Snakemake](https://snakemake.readthedocs.io)) rules require a substantial amount of RAM memory, forcing the OS to abruptly terminate the process and show the cryptic message `Killed`.

3. Once logged in one of GenomeDK back-end machines, go in directory `workspace/pypsa-eur` by executing the following:

    ```bash
    username@genomedkbackend:~$ cd workspace/pypsa-eur
    ```

4. Activate the Anaconda environment tailored for PyPSA-Eur (created in step `14` of the [installation](#install) procedure) by executing the following:

    ```bash
    username@genomedkbackend:~/workspace/pypsa-eur$ conda activate pypsa-eur
    ```

5. Copy PyPSA-Eur default configuration file `config.default.yaml` (found in PyPSA-Eur directory `config`) to file `config.yaml` by executing the following:

    ```bash
    username@genomedkbackend:~/workspace/pypsa-eur$ cp config/config.default.yaml config/config.yaml
    ```

6. Set the relevant configuration options in file `config.yaml` (found in PyPSA-Eur directory `config`) with appropriate values representing the model to run in GenomeDK - see file [config.default.yaml](https://github.com/PyPSA/pypsa-eur/blob/master/config/config.default.yaml) to get an enumeration of the configuration options available (a thorough explanation about these can be found [here](https://pypsa-eur.readthedocs.io/en/latest/configuration.html)). Some configuration options that are usually set with new values are `run:name`, `scenario:ll`, `scenario:clusters` and `scenario:sector_opts`, to name a few. In addition, setting configuration option `run:shared_resources` to `true` may be a good idea to avoid downloading and processing the same resources every time a new run is launched (otherwise the run will take additional time to complete).


7. Run PyPSA-Eur model based on the configuration done in step `6` by executing the following (this might take a while to complete, especially the first time it runs):

    ```bash
    username@genomedkbackend:~/workspace/pypsa-eur$ snakemake -call all -j1
    ```


## Tips & Tricks

1. To avoid having to provide user credentials every time a connection to GenomeDK is made, setting up a public key authentication in this machine is a good idea. First, generate a public key using a tool named [ssh-keygen](https://en.wikipedia.org/wiki/Ssh-keygen) by executing the following (press ENTER when asked questions by `ssh-keygen` as the default values are appropriate for this case):

    ```bash
    privateusername@privatemachine:~$ ssh-keygen
    ```

   After generating a public key, copy it to GenomeDK by executing the following (replace `<username>` with the information provided upon requesting an account in step `1` of the [installation](#install) procedure):

    ```bash
    privateusername@privatemachine:~$ sh-copy-id -i ~/.ssh/id_rsa <username>@login.genome.au.dk
    ```

   From this point on, every time a connection is made to GenomeDK - through, e.g., `ssh` or `scp` - no user credentials are asked and the connection will be made transparently.


2. To copy files from a machine to GenomeDK, the tool [scp](https://en.wikipedia.org/wiki/Secure_copy_protocol) may be useful in this task due to its simplicity and ubiquity in all Linux distributions. Therefore, to copy a file to GenomeDK execute the following (replace `<file>` with the name of the file to copy, `<username>` with the information provided upon requesting an account in step `1` of the [installation](#install) procedure and `<location>` with the location in the user home in GenomeDK where the file will be copied to):

    ```bash
    privateusername@privatemachine:~$ scp <file> <username>@login.genome.au.dk:~/<location>
    ```

   Moreover, in case of need to copy an entire directory (including sub-directories) execute the following (replace `<directory>` with the name of the directory to copy, `<username>` with the information provided upon requesting an account in step `1` of the [installation](#install) procedure and `<location>` with the location in the user home in GenomeDK where the directory will be copied to):

    ```bash
    privateusername@privatemachine:~$ scp -rp <directory> <username>@login.genome.au.dk:~/<location>
    ```

3. When performing a long-running process (or task), e.g. a PyPSA-Eur model, on a remote machine, e.g. GenomeDK, the capability to persist a session when the connection to the machine drops for whatever the reason becomes important (otherwise all the work done by the process in the meantime will be lost). To that end, a tool named [screen](https://en.wikipedia.org/wiki/GNU_Screen) is very useful given its capability of persisting a session to a remote machine even when the connection to it is no longer available. To persist a session using `screen`, execute the following:

    ```bash
    username@genomedkbackend:~$ screen
    ```

   This will make `screen` to be in the background and return to the terminal. The user may then launch a long-running process and press `Ctrl+a` followed by `d` afterwards. Pressing this keys sequence will: 1) detach the process, 2) display information about the detachment (e.g. `23625.pts-0.username (04/28/2023 09:36:05 AM) (Attached)`) and 3) return to the terminal. The session is preserved now even when the connection to the remote machine is lost. To reattach to the session (i.e. to see it again), log in to the remote machine and execute the following (replace `<session_id>` with the number displayed upon detaching the session - e.g. `23625`):

    ```bash
    username@genomedkbackend:~$ screen -r <session_id>
    ```


## Support

The present notes are actively maintained by the Energy Systems Group at [Aarhus University](https://www.au.dk) (Denmark). Please open a ticket [here](https://github.com/ricnogfer/pypsa-in-genomedk/issues) in case something is missing, incomplete, outdated or incorrect.


