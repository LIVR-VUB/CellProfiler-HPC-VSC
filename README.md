# CellProfiler on HPC: A Complete Guide

This repository provides tools, scripts, and documentation for installing, configuring, and running [CellProfiler](https://cellprofiler.org/) in high-performance computing (HPC) environments. It is designed to support large-scale image analysis workflows, including Cell Painting and high-content screening, on SLURM-based clusters.

## Repository Contents

- `scripts/` – Python scripts for executing CellProfiler in parallel
- `nf1_analysis.py` – Main script to orchestrate plate-wise analysis
- `cp_parallel.py` – Logic for multiprocessing CellProfiler runs
- `cp_sequential.py` – Post-processing script to rename output files
- `slurm_scripts/` – SLURM submission templates for different configurations
- `README.md` – Setup and usage instructions

---

## Installation on Ubuntu 22.04 with Python 3.8

These instructions guide you through installing CellProfiler and its dependencies on Ubuntu 22.04.

### Step 1: Update the system

```bash
sudo apt update
sudo apt -y upgrade
````

### Step 2: Install required dependencies

```bash
sudo apt install -y build-essential python3-dev default-libmysqlclient-dev openjdk-11-jdk-headless libgtk-3-dev libnotify-dev libsdl2-dev
```

### Step 3: Set environment variables for Java

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:/home/ubuntu/.local/bin
```

To make these permanent, add the above lines to your `.bashrc` or `.zshrc`:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:/home/ubuntu/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

For zsh:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.zshrc
echo 'export PATH=$PATH:/home/ubuntu/.local/bin' >> ~/.zshrc
source ~/.zshrc
```

### Step 4: Install wxPython

```bash
pip install https://extras.wxpython.org/wxPython4/extras/linux/gtk3/ubuntu-22.04/wxPython-4.2.0-cp38-cp38-linux_x86_64.whl
```

### Step 5: Install CellProfiler

```bash
pip install cellprofiler
```

### Optional: Use a Python virtual environment

To avoid package conflicts:

```bash
pip install virtualenv
virtualenv --python=python3.8 cellprofiler_env
source cellprofiler_env/bin/activate
```

Then re-run the steps above within the virtual environment.

---

## HPC Usage Instructions from Institutional Guides

### BioHPC at UT Southwestern

Guide: [CellProfiler on BioHPC](https://portal.biohpc.swmed.edu/content/guides/cellprofiler-biohpc/)

Key steps:

* Load CellProfiler and Anaconda modules:

  ```bash
  module load anaconda/2020.11
  module load cellprofiler/4.2.1
  ```
* Use the provided SLURM template for job submission.

### Rivanna HPC at University of Virginia

Guide: [CellProfiler on Rivanna](https://www.rc.virginia.edu/userinfo/hpc/software/cellprofiler/)

Key steps:

* Load the Apptainer module:

  ```bash
  module load apptainer
  ```
* Use CellProfiler in containerized form:

  ```bash
  apptainer exec $HOME/cellprofiler/cp-4.2.5.sif cellprofiler -c -r -p pipeline.cppipe -i images/ -o output/
  ```

### Hydra HPC at VUB

Key steps:

* Apptainer module: *Currently under development* - LIVR

* We can run a complete pipeline by making a *conda* environment with these requirements: https://github.com/WayScience/nf1_schwann_cell_painting_data/blob/main/environments/nf1_cellpainting_env.yml

* Suggested partition config in Hydra: 2X *Zen4: AMD EPYC 9384X* (64 cores) + 64GB RAM (minimum)
* ### Important: More here on monitoring *CellProfiler* performance on **Hydra** - (https://github.com/arka2696/CellProfiler-HPC/blob/main/CellProfilerHPC-Performance-Check.md)


---

## Example SLURM Script for VUB HPC

```bash
#!/bin/bash
#SBATCH --job-name=cellprofiler
#SBATCH --partition=zen4
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=64
#SBATCH --mem=64G
#SBATCH --time=72:00:00
#SBATCH --output=logs/%x_%j.log

# This path should be chnaged based on your needs
cd /scratch/brussel/vo/000/bvo00026/vsc11013/Imaging/cytomining/nf1_schwann_cell_painting_data/2.cellprofiler_analysis  

module purge
module load Mamba
source $EBROOTMAMBA/etc/profile.d/conda.sh
conda activate nf1_cellpainting_data

python scripts/nf1_analysis.py
```

---

## Best Practices for HPC Deployment

* Use job arrays for parallel plate-wise processing
* Assign 8–16 cores per plate, depending on workload
* Prefer local scratch storage for input/output
* Monitor CPU and memory usage with `sacct` or `htop`
* Consider limiting thread overuse with `OMP_NUM_THREADS` and `OPENBLAS_NUM_THREADS`

---

## License

This repository is distributed under the MIT License.

---

## Contributing

Contributions are welcome. Please open an issue or submit a pull request to discuss changes or additions.

---

## Contact

If you have questions or suggestions, please use the GitHub issues page or email the repository maintainer.

```

---
