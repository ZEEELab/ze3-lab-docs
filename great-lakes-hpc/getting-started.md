# Getting started on the Great Lakes HPC Cluster

Contents
<!-- TOC -->

- [Resources](#resources)
- [Getting logged on](#getting-logged-on)
- [Getting access to the lab's job submission queue + scratch space](#getting-access-to-the-labs-job-submission-queue--scratch-space)
- [Submitting your first job](#submitting-your-first-job)
  - [A basic job](#a-basic-job)
  - [An array job](#an-array-job)
- [Downloading data from Great Lakes](#downloading-data-from-great-lakes)
  - [Using sftp (command line)](#using-sftp-command-line)

<!-- /TOC -->

## Resources

- [Great Lakes cheat sheet](./Great-Lakes-Cheat-Sheet.pdf)
- [Great Lakes user guide](https://arc.umich.edu/greatlakes/user-guide/)
- [Slurm user guide for Great Lakes](https://arc.umich.edu/greatlakes/slurm-user-guide/)
- [Slurm documentation](https://slurm.schedmd.com/documentation.html)
- [Bash scripting cheat sheets](https://devhints.io/bash)

## Getting logged on

1. **Request a login!** Find a link to the login form at the top of ITS's [user guide](https://arc.umich.edu/greatlakes/user-guide/).
   - As part of the request, you'll need to provide your department, advisor, a reason for the request (e.g., you want to do cool digital evolution experiments), and which service you want a login for (GreatLakes in this case).
2. **Logging on**
   - Via a terminal (ssh)
     - For other ways of logging on and interacting with the HPC, see the [user guide](https://arc.umich.edu/greatlakes/user-guide/)
     - Run `ssh username@greatlakes.arc-ts.umich.edu` where `username` is replaced with your UM unique id
     - This will prompt you for your password, and then require two-factor authentication (e.g., DUO).
     - Note that if you're not on campus WiFi, you'll need to be on UM's VPN. [Guide here](https://its.umich.edu/enterprise/wifi-networks/vpn/getting-started).
3. Once you've logged on (assuming you're accessing the HPC via ssh), you should be inside your home directory. As of me writing this, your home directory is limited to 80gb of storage (run `home-quota` to seee where you're space-wise).

## Getting access to the lab's job submission queue + scratch space

1. **Ask Luis** to request that your user account is added to the lab's accounts (at the time of writing this, `zamanlh0` and `zamanlh1`)
2. Once the request is approved, if you run `my_accounts`, you should see the `zamanlh0` and `zamanlh1` in the list. If so, you now have access to `/scratch/zamanlh_root/`, and you can submit jobs with either the `zamanlh0` or `zamanlh1` account.
   - Inside `/scratch/zamanlh_root/`, you should see the following directories: `zamanlh0` or `zamanlh1`
   - Inside each of `zamanlh0` and `zamanlh1`, you should have your own directory (e.g., mine is `lalejini`), and you have access to the `shared_data` directory. The `shared_data` directory is accessible to everyone in the lab, so it's a great place to put data that you want others to be able to work with. This also means that its possible for you to overwrite someone else's data inside the `shared_data` directory, so be mindful!

## Submitting your first job

Okay, you've got a great lakes login, and you've got access to the lab's job queue. Now what? You can do what you came here for: run your software (e.g., experiments, analyses, etc.) on the HPC.

In general, you're going to want to run your code as a job. There are a few different types of jobs ([see here](https://arc.umich.edu/greatlakes/slurm-user-guide/)). For any job that you want to run, you need to ask the HPC to schedule it for you.
Unlike your local machine, there are lots of users vying for the HPC's resources, and you need to put in a request to get scheduled to use those resources.
Great Lakes uses [slurm](https://slurm.schedmd.com/documentation.html) to schedule things and manage resources.

Great lakes has a useful guide for submitting jobs: <https://arc.umich.edu/greatlakes/slurm-user-guide/#interactive-jobs>.

I'll throw two annotated examples below.

### A basic job

You submit jobs in the form of batch scripts. Here's an example script that requests a single CPU on a single node. Each batch script is basically a fancy [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) script with some special comments (that specify request information) at the top.

Here's an example batch script (e.g., maybe called `test.sh`).

```
#!/bin/bash
# The line above this specifies interpreter used to execute the script.
# Except for that first line & the '#SBATCH' lines, everything behind a '#' is a comment (and isn't executed).

# These next 7 lines specify how your job should be scheduled (what resources you want, etc).
#SBATCH --job-name=test_job
#SBATCH --mail-type=FAIL
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=1000m
#SBATCH --time=00:01:00
#SBATCH --account=zamanlh0

# -- I like to define helpful variables up top --
USERNAME=lalejini
OUTPUT_DIR=/scratch/zamanlh_root/zamanlh0/${USERNAME}

echo "hello world" > ${OUTPUT_DIR}/hello.txt
```

Checkout the [Great Lakes slurm user guide](https://arc.umich.edu/greatlakes/slurm-user-guide/#interactive-jobs) for what each of the `#SBATCH` parameters do! In summary, this example requests a single CPU with a 1000mb allocation of RAM for 1 minute. It will get queued with the `zamanlh0` account. Unfortunately, compute time is not free on Great Lakes, so chat with Luis about what sorts of resources we have access to.

You use the `sbatch` command to submit jobs to be scheduled. In this case (assuming the example is in a script called `test.sh`), you would run `sbatch test.sh`.

### An array job

For digital evolution experiments, we often want to run many replicates, each using the same program (e.g., Avida), but each independent run/replicate should have its own random number seed.

Array jobs are often a good way to accomplish this sort of setup.

```
#!/bin/bash
# The interpreter used to execute the script

#SBATCH --job-name=test_job
#SBATCH --mail-type=FAIL
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --mem=1000m
#SBATCH --time=00:01:00
#SBATCH --account=zamanlh0
#SBATCH --array=1-10

# -- I like to define helpful variables up top --
USERNAME=lalejini
OUTPUT_DIR=/scratch/zamanlh_root/zamanlh0/${USERNAME}
JOB_ID=${SLURM_ARRAY_TASK_ID}

echo "hello world" > ${OUTPUT_DIR}/job-${JOB_ID}/hello.txt
```

Notice that this script is just slightly different from the previous example. We've added `#SBATCH --array=1-10` to specify that this is an array job, and we're using the `SLURM_ARRAY_TASK_ID` environment variable to set a job id. This array job will create 10 independent jobs, and each will create a job-X directory (where X is specified by the job's array id) and make a `hello.txt` files inside that directory.

## Downloading data from Great Lakes

### Using sftp (command line)

"sftp" = [Secure File Transfer Protocol](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol)

1. Log on to great lakes with sftp: `sftp username@greatlakes-xfer.arc-ts.umich.edu` where `username` is replaced by your username.
   - Once you've logged on successfully, your command line prompt should be something like `sftp>`
2. You are now logged onto the HPC via sftp. You can navigate the file system much like you would as if you were ssh'd into the HPC. Use the `get` command to download files from the HPC to your local machine. (ctrl-f for `get` in [this sftp documentation](https://man7.org/linux/man-pages/man1/sftp.1.html))
   - For example, if I wanted to download a file called `data.csv` at `/scratch/zamanlh_root/zamanlh0/lalejini`, I could run `get /scratch/zamanlh_root/zamanlh0/lalejini/data.csv` to download `data.csv`.
   - If I wanted to download an entire directory (including the content of all of its subdirectories), I could use the `-r` argument for `get`. For example, `get /scratch/zamanlh_root/zamanlh0/lalejini/data` would download _everything_ inside of the `/scratch/zamanlh_root/zamanlh0/lalejini/data` directory.




