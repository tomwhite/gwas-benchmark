## Setup

- Create an `n1-standard-8` GCE instance w/ Debian 10 (buster) OS
- [Install conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html)
- Install extra packages
```bash
sudo apt-get install -y git ntp
```
- Create a conda environment
```bash
conda env create -f envs/gwas-benchmark.yaml 
conda activate gwas-benchmark
```