# Virtuoso: Enabling Fast and Accurate Virtual Memory Research via an Imitation-based Operating System Simulation Methodology

This repository provides all the necessary files and instructions to reproduce the results of our ASPLOS 2025 paper.

> Konstantinos Kanellopoulos, Konstantinos Sgouras, F. Nisa Bostanci, Andreas Kosmas Kakolyris, Berkin K. Konar, Rahul Bera, Mohammad Sadrosadati, Rakesh Kumar, Nandita Vijaykumar, and Onur Mutlu, "Virtuoso: Enabling fast and accurate virtual memory research via an imitation-based OS simulation methodology," ASPLOS'25. [Paper PDF](https://arxiv.org/pdf/2403.04635v2)

Please use the following citation if this repository is useful for your work:

```bibtex
@inproceedings{kanellopoulos2025virtuoso,
    title={{Virtuoso: Enabling Fast and Accurate Virtual Memory Research via an Imitation-based Operating System Simulation Methodology}},
    author={Konstantinos Kanellopoulos, Konstantinos Sgouras, F. Nisa Bostanci, Andreas Kosmas Kakolyris, Berkin K. Konar, Rahul Bera, Mohammad Sadrosadati, Rakesh Kumar, Nandita Vijaykumar, and Onur Mutlu},
    year={2025},
    booktitle={ASPLOS}
}
```

## Website & Documentation

- Please visit our [website](https://cmu-safari.github.io/Virtuoso) to find tutorials and documentation on how to use Virtuoso.

## Structure of the Repository

1. **Introduction**
     - Describes the motivation of this work and introduces Virtuoso
2. **Prerequisites**
     - Describes the prerequisites for running the experiments
3. **Running Experiments**
     - Provides instructions on how to run a test experiment using the provided scripts
4. **Datasets**
     - Provides information on the datasets

## Introduction

Virtuoso is a new simulation framework designed to enable fast and accurate prototyping and evaluation of virtual memory (VM) schemes.  It employs a lightweight userspace kernel, MimicOS, which imitates the desired OS kernel code, allowing researchers to simulate only the relevant OS routines and easily develop new OS modules.  Virtuoso's imitation-based OS simulation methodology facilitates the evaluation of hardware/OS co-designs by accurately modeling the interplay between OS routines and hardware components. 

## Hardware Requirements

- **Architecture**: x86-64 system.
- **Memory**: 4–13 GB of free memory per experiment.
- **Storage**: 10 GB of storage space for the dataset.


## Software Requirements

- Python 3.8 or later (Virtuoso+Sniper can also work with Python 2.7 but we do not recommend it)
- libpython3.8-dev or later (make sure to install the correct version of libpython for your Python version)
- g++


We provide a script to install some potentially missing dependencies. 
The will also install Miniconda, which is a lightweight version of Anaconda so that you can create a virtual environment for Python 3.8 or later.

```bash
cd Virtuoso/simulator/sniper/
sh install_dependencies.sh
bash # start a new bash shell to make sure the environment variables are set
conda create -n virtuoso python=3.10 
conda activate virtuoso # Python 3.10 will be your default Python version in this environment
```
Download some traces from workloads that experienced high address translation overheads. 

```bash
sh download_traces.sh
```
## Getting Started

 
Let's first clean the Sniper simulator and then build it from scratch.

```bash
cd Virtuoso/simulator/sniper
make distclean # clean up so that we can build from scratch
make -j # build Sniper
```

Make sure the simulator is working by running the following command:

```bash
sh run_example.sh 
```

Let's now do something more interesting and run actual experiments. 
We will run the following experiment:

1) Baseline MMU with a Radix page table and a reservation-based THP policy (similar to the one used in FreeBSD) with 4KB and 2MB pages.
2) We will sweep the memory fragmentation ratio to observe the effect of memory fragmentation on the performance of address translation.
3) We will run the experiment multiple translation-intensive workloads.

To run the experiments efficiently, we will use the Slurm job scheduler. 
If you do not have Slurm installed, you can run the experiments by modifying the `create_jobfile_virtuoso_reservethp.py` script to run the experiments sequentially without `sbatch` and `srun`.

```bash
cd Virtuoso/scripts/virtuoso_sniper
python3 create_jobfile_virtuoso_reservethp.py ../../Virtuoso/ ../jobfiles/reservethp.jobfile
cd ../jobfiles
source reservethp.jobfile # This will run the experiments with Slurm
```


## Release Notes

### **2025-07: Planned 3rd Major Update**
- **New Features:**
    - Integration of MimicOS with a [CXL simulator](https://github.com/Amit-P89/-DRackSim/)
    - Integration of MimicOS with a GPU simulator.
    - Virtuoso+Sniper with support for IOMMU.
    - MimicOS with network stack support.

---

### **2025-05: Planned 2nd Major Update**
- **New Features:**
    - Release of all MimicOS modules, including:
        - hugetlbfs
        - Page cache
        - Swap cache
          
    - Integration of MimicOS with:
        - [gem5-SE](http://gem5.org/)
        - [Ramulator](https://github.com/CMU-SAFARI/ramulator2)
        - [Sniper](https://github.com/snipersim/)
        - [ChampSim](https://github.com/ChampSim/ChampSim)
          
    - New traces/workloads with address translation overheads:
        - MemCached
        - Redis
        - Stockfish
          
    - Release of memory tagging schemes

---

### **2025-04-02: Initial Release**
- **Virtuoso Integration:**
    - [Sniper Multi-Core Simulator] (https://github.com/snipersim/)

- **MMU Models:**
    1. **MMU Baseline:**
         - Page Walk Caches
         - Configurable TLB hierarchy.
         - Configurable Page Walk Cache (PWC) hierarchy
         - Large page prediction based on [Papadopoulou et al.](https://ieeexplore.ieee.org/document/7056034)
    2. **MMU Speculation:** Speculative address translation as described in [SpecTLB](https://ieeexplore.ieee.org/document/6307767)
    3. **MMU Software-Managed TLB:** Software-managed L3 TLB as described in [POM-TLB](https://ieeexplore.ieee.org/document/8192494)
    4. **MMU Utopia:** Implements [Utopia](https://arxiv.org/abs/2211.12205)
    5. **MMU Midgard:** Implements [Midgard](https://dl.acm.org/doi/10.1109/ISCA52012.2021.00047)
    6. **MMU RMM (and Direct Segments):** Implements [RMM](https://scail.cs.wisc.edu/papers/isca15-rmm.pdf)
    7. **MMU Virtualized:** Nested Paging and Nested Page Tables (NPT) for modern hypervisors

- **Page Table Designs:**
    1. **Page Table Baseline:** Radix page table with configurable page sizes
    2. **Range Table:** B++ Tree-like translation table for [virtual-to-physical address ranges](https://scail.cs.wisc.edu/papers/isca15-rmm.pdf)
    3. **Hash Don't Cache:** [Open-addressing hash-based page table](https://dl.acm.org/doi/10.1145/2964791.2901456)
    4. **Conventional Hash-Based:** Chain-based hash table design
    5. **ECH:** [Cuckoo hashing-based organization of the page table](https://iacoma.cs.uiuc.edu/iacoma-papers/asplos20.pdf)
    6. **RobinHood:** Open-addressing with element re-ordering

- **Memory Allocation Policies:**
    1. **Reservation-Based THP:** Implements reservation-based Transparent Huge Pages
    2. **Eager-Paging:** Implements the contiguity-based allocation described in [RMM](https://scail.cs.wisc.edu/papers/isca15-rmm.pdf)
    3. **Utopia:** Implements the allocation mechanism described in [Utopia](https://arxiv.org/abs/2211.12205)


## Contributing
We welcome contributions! Please open an issue on GitHub to discuss potential changes or report bugs.

## Contact 
For questions please contact Konstantinos Kanellopoulos (<konkanello@gmail.com>) and Konstantinos Sgouras (<sgouraskon@gmail.com>).
