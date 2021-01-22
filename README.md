## Setup

- Create an `n1-standard-8` GCE instance w/ Debian 10 (buster) OS with 1000GB disk, and full access to all Cloud APIs
- [Install conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html)
```bash
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# follow prompts then reconnect
```
- Install extra packages
```bash
sudo apt-get install -y git ntp
```
- Clone this repo
```bash
git clone https://github.com/tomwhite/gwas-benchmark
cd gwas-benchmark
```
- Create a conda environment
```bash
conda env create -f envs/gwas-benchmark.yaml 
conda activate gwas-benchmark
```
- Start Jupyter
```bash
jupyter notebook
```
- Create a proxy from your local machine
```bash
gcloud beta compute ssh --zone "us-central1-a" "ukb-tw-gwas-benchmark" --ssh-flag="-L 8888:localhost:8888"
```
- Open the Jupyter URL

## Other

Copy notebook from VM back to local machine 
```bash
gcloud beta compute scp --zone "us-central1-a" "ukb-tw-gwas-benchmark:/home/tom/gwas-benchmark/gwas_simulation.ipynb" .
```

Create a proxy for the Dask UI (when in distributed mode)
```bash
gcloud beta compute ssh --zone "us-central1-a" "ukb-tw-gwas-benchmark" --ssh-flag="-L 8799:localhost:8787"
```

Copy performance report from VM back to local machine and open latest
```bash
gcloud beta compute scp --zone "us-central1-a" "ukb-tw-gwas-benchmark:/home/tom/gwas-benchmark/reports/pr_*.html" reports
open $(ls -t reports/pr*.html | head -1)
```

### Results

Running XY-size on n1-standard-8

| Dask  | Time (s) |
| ----- | -------- |
| In-memory, local storage  | 30 |
| In-memory, GCS  | 70 |
| Distributed (1 worker), local storage  | 43 |
| Distributed (1 worker), GCS  | 95 |

The distributed/GCS timing is roughly similar to the actual GWAS, which took 80s.
