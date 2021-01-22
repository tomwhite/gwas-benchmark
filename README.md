# GWAS Benchmark

A simulated GWAS benchmark running on Dask to test scaling.

Background:
- [Identify lack of scalability in gwas_linear_regression](https://github.com/pystatgen/sgkit/issues/390)
- [Estimate cost of GWAS regression steps](https://github.com/related-sciences/ukb-gwas-pipeline-nealelab/issues/32)
- [Create a public GWAS workflow for collaborative optimization](https://github.com/pystatgen/sgkit/issues/438)

This repo uses some of the supporting code from https://github.com/related-sciences/ukb-gwas-pipeline-nealelab.

## Overview

To run the benchmark notebook, the following infrastructure is used:

1. A development [GCE](https://cloud.google.com/compute) VM
2. Dask clusters
    - These will be managed using [Dask Cloud Provider](https://cloudprovider.dask.org/en/latest/)

The development VM is used to issue the GWAS code that run on the Dask clusters. In some cases no Dask cluster is
needed since a (small) Dask cluster is run on the development VM.

## Setup

- Check out this repo on your local machine.
- Create a `.env` file in the base of the checked-out repo. It contains more sensitive variable settings and a prototype for this file is shown here:
```
export GCP_PROJECT=XXXXX
export GCP_REGION=us-east1
export GCP_ZONE=us-east1-c
export GCS_BUCKET=my-bucket-name # An existing bucket is required to store test data in
export GCP_USER_EMAIL=XXXXXXXXXXXX-compute@developer.gserviceaccount.com # GCP user to be used in ACLs
export GCE_WORK_HOST=gwas-benchmark # Hostname given to development VM
```
- Create an `n1-standard-8` GCE instance w/ Debian 10 (buster) OS with 1000GB disk, and full access to all Cloud APIs. You can do this manually through the web console, or by running a command like the following:
```bash
source .env
gcloud beta compute --project=$GCP_PROJECT instances create $GCE_WORK_HOST \
    --zone=$GCP_ZONE \
    --machine-type=n1-standard-8 \
    --subnet=default \
    --network-tier=PREMIUM \
    --maintenance-policy=MIGRATE \
    --service-account=$GCP_USER_EMAIL \
    --scopes=https://www.googleapis.com/auth/cloud-platform \
    --image=debian-10-buster-v20201216 \
    --image-project=debian-cloud \
    --boot-disk-size=1000GB \
    --boot-disk-type=pd-standard \
    --boot-disk-device-name=$GCE_WORK_HOST \
    --no-shielded-secure-boot \
    --shielded-vtpm \
    --shielded-integrity-monitoring \
    --reservation-affinity=any
```
- When the development VM is running, connect to it with
```bash
gcloud beta compute ssh --zone $GCP_ZONE $GCE_WORK_HOST
```
- Install required software
```bash
sudo apt-get update && sudo apt-get install -y git ntp wget
# https://docs.anaconda.com/anaconda/install/silent-mode/#linux-macos
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p $HOME/miniconda
~/miniconda/bin/conda init bash
exec bash
```
- Clone this repo
```bash
git clone https://github.com/tomwhite/gwas-benchmark && cd gwas-benchmark
```
- Install `.env` (run from local machine)
```bash
source .env
gcloud beta compute scp --zone $GCP_ZONE .env "$GCE_WORK_HOST:gwas-benchmark/.env"
```
- Create a conda environment
```bash
source .env
conda env create -f envs/gwas-benchmark.yaml 
conda activate gwas-benchmark
```
- Start Jupyter
```bash
jupyter notebook
```
- Create a proxy from your local machine
```bash
source .env
gcloud beta compute ssh --zone $GCP_ZONE $GCE_WORK_HOST --ssh-flag="-L 8888:localhost:8888"
```
- Open the Jupyter URL in a browser and open the notebook.

## Running a Dask cluster

- Connect to the development machine
```bash
source .env
gcloud beta compute ssh --zone $GCP_ZONE $GCE_WORK_HOST
```
- Authenticate
```bash
gcloud auth login
```
- Follow the instructions in https://github.com/related-sciences/ukb-gwas-pipeline-nealelab#dask-cloud-provider
- Create a proxy for the Dask UI (substitute the name of your scheduler instance)
```bash
gcloud beta compute ssh --zone $GCP_ZONE dask-06d1dd24-scheduler --ssh-flag="-L 8799:localhost:8787"
```

## Miscellaneous useful commands

Create a proxy for the Dask UI (when in distributed mode on the development machine, not the cloudprovider cluster)
```bash
gcloud beta compute ssh --zone $GCP_ZONE $GCE_WORK_HOST --ssh-flag="-L 8799:localhost:8787"
```

Copy performance report from VM back to local machine and open latest
```bash
gcloud beta compute scp --zone $GCP_ZONE "$GCE_WORK_HOST:gwas-benchmark/reports/pr_*.html" reports
open $(ls -t reports/pr*.html | head -1)
```

Copy the notebook from VM back to local machine (so it can be checked in to source control)
```bash
gcloud beta compute scp --zone $GCP_ZONE "$GCE_WORK_HOST:gwas-benchmark/gwas_simulation.ipynb" .
```

# Results

Running XY-size on n1-standard-8 instances

| Dask  | Time (s) | Notes |
| ----- | -------- | ----- |
| In-memory, local storage  | 30 | |
| In-memory, GCS  | 70 | |
| Distributed (1 worker), local storage  | 43 | |
| Distributed (1 worker), GCS  | 95 | similar to the actual GWAS, which took 80s |
| Distributed (4 workers), GCS  | 41 | 4x cluster gives ~2x speedup |
