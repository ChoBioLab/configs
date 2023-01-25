# nf-core/configs: Icahn School of Medicine at Mount Sinai

To use, run the pipeline with `-profile mssm`. This will download and launch the [`mssm.config`](../conf/mssm.config) which has been pre-configured with a setup suitable for the Minerva HPC environment.

## Running the workflow on the Minerva HPC

The latest version of Nextflow is not installed by default on the cluster. You will need to install it into a directory you have write access to.

The template script below relies on a conda install for Nextflow. Following this conda installation will add the necessary package channels, setup a new env named "nextflow", and install Nextflow.

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda create --name nextflow
conda activate nextflow
conda install nextflow
```

## Master execution script

Nextflow manages each process as a separate job that is submitted to the cluster by using the `bsub` command. To do so, make a single, master script with a similar structure to the following code, and submit with `bsub < my_script.lsf` to orchestrate the full slate of tasks in the pipeline.

```bash
#!/bin/bash
#BSUB -J <JOB TITLE>
#BSUB -P <PROJECT NAME>
#BSUB -W 144:00
#BSUB -q premium
#BSUB -n 2
#BSUB -u <EMAIL>
#BSUB -o output_%J.stdout
#BSUB -eo error_%J.stderr
#BSUB -L /bin/bash

export http_proxy=http://172.28.7.1:3128
export https_proxy=http://172.28.7.1:3128
export all_proxy=http://172.28.7.1:3128
export no_proxy=localhost,*.chimera.hpc.mssm.edu,172.28.0.0/16

ml java
ml anaconda3
ml singularity

source $CONDA_PREFIX/etc/profile.d/conda.sh
conda init bash
conda activate nextflow

nextflow run <PIPELINE> \
	-w /sc/arion/scratch/$USER \
	-profile mssm \
	-resume

```
