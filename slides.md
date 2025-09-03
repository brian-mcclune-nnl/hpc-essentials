---
marp: true
theme: default
paginate: true
backgroundColor: #ffffff
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
---

<!-- _class: lead -->

# HPC Essentials
## A Guide for Data Scientists and Engineers

**Connecting • Computing • Conquering**

---

## Table of Contents

1. **Introduction to HPC**
2. **Connecting to HPC Systems**
3. **Unix/Linux Fundamentals** 
4. **Python on HPC**
5. **SLURM Job Management**
6. **Best Practices & Next Steps**

---

<!-- _class: lead -->

# 1. Introduction to HPC

---

## What is High Performance Computing?

- **Parallel processing** across multiple compute nodes
- **Shared resources** managed by job schedulers
- **Specialized hardware** (CPUs, GPUs, high-speed interconnects)
- **Designed for** computationally intensive workloads

### Why HPC for Data Science?
- Large datasets that don't fit in memory
- Complex models requiring significant compute time
- Parallel processing of independent tasks
- Access to specialized hardware (GPUs, large memory nodes)

---

<!-- _class: lead -->

# 2. Connecting to HPC Systems

---

## Connection Methods

### SSH (Secure Shell)
**From Terminal/Command Line:**
```bash
ssh username@hpc-cluster.institution.gov
ssh -A username@cluster.gov      # Agent forwarding
ssh -AY username@cluster.gov     # Agent + trusted X11 forwarding
```

### PuTTY-CAC (Government Windows)
- **PuTTY-CAC**: Enhanced PuTTY for government CAC authentication
- **GitHub**: https://github.com/NoMoreFood/putty-cac
- **Windows OpenSSH Integration**: Via PuTTY-CAC's Pageant
- **Agent Forwarding**: Built-in support for SSH agent forwarding

---

## SSH Best Practices

### Key-Based Authentication
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your_email@gov.agency"

# Add public key to authorized_keys (shared /home filesystem)
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

---

## Government SSH Configuration

### Command Line Options
```bash
# Basic connection
ssh username@cluster.gov

# With agent forwarding
ssh -A username@cluster.gov

# Agent + trusted X11 forwarding
ssh -AY username@cluster.gov
```

### Why Use These Options?
- **`-A`**: Forward SSH agent (access other nodes without re-auth)
- **`-Y`**: Trusted X11 forwarding (run GUI applications)

---

## SSH Config File

### Create `~/.ssh/config` for convenience:
```
Host mycluster
    HostName hpc-cluster.institution.gov
    User myusername
    IdentityFile ~/.ssh/id_rsa
    ForwardAgent yes              # Enable agent forwarding
    ForwardX11 yes               # Enable X11 forwarding
    ForwardX11Trusted yes        # Trusted X11 forwarding
```

### Usage
```bash
# Instead of: ssh -AY myusername@hpc-cluster.institution.gov
ssh mycluster    # Much simpler!
```

---

<!-- _class: lead -->

# 3. Unix/Linux Fundamentals

---

## Shell Basics

### What is a Shell?
- **Interface** between user and operating system
- **Command interpreter** that executes your commands
- **Scripting environment** for automation

### Common Shell Flavors
- **bash**: Bourne Again Shell (most common)
- **zsh**: Z Shell (enhanced bash)
- **csh/tcsh**: C Shell family (**default on our HPC systems**)
- **fish**: Friendly Interactive Shell

---

## Important Unix Concepts

### Case Sensitivity
```bash
# These are all different!
MyFile.txt
myfile.txt
MYFILE.TXT
```

### Spaces in Filenames
```bash
# Problematic
my file.txt

# Solutions
"my file.txt"    # Quotes
my\ file.txt     # Escape
my_file.txt      # Underscores (recommended)
```

---

## Essential Commands - Navigation

<div class="columns">

<div>

### Directory Operations
```bash
pwd             # Print working directory
ls              # List files
ls -la          # Detailed listing
cd /path/to/dir # Change directory
cd ~            # Go to home
cd ..           # Go up one level
mkdir dirname   # Create directory
rmdir dirname   # Remove empty directory
```

</div>

<div>

### File Operations
```bash
touch file.txt  # Create empty file
cp file1 file2  # Copy file
cp -r dir1 dir2 # Copy directory
mv old new      # Move/rename
rm file.txt     # Remove file
rm -rf dirname  # Remove directory
ln -s target link # Symbolic link
```

</div>

</div>

---

## Paths: Absolute vs Relative

### Absolute Paths
```bash
/home/username/data/results.csv
/usr/local/bin/python
/tmp/output.log
```
- Start with `/`
- Complete path from root
- Work from anywhere

### Relative Paths
```bash
data/results.csv       # Relative to current directory
../other_project/      # Go up one level, then down
./script.py           # Current directory (explicit)
```

---

## File Permissions

### Understanding Permissions
```bash
-rwxr-xr-- 1 user group 1024 Jan 1 12:00 myfile.txt
```

- **First character**: File type (`-` file, `d` directory, `l` link)
- **Next 9 characters**: Permissions (owner, group, others)
  - `r`: read (4), `w`: write (2), `x`: execute (1)

### Changing Permissions
```bash
chmod 755 script.py    # rwxr-xr-x
chmod +x script.py     # Add execute permission
chmod u+w,g-w file.txt # User add write, group remove write
```

---

## Essential Commands - Text Processing

<div class="columns">

<div>

### Viewing Files
```bash
cat file.txt      # Display entire file
head file.txt     # First 10 lines
head -n 20 file   # First 20 lines
tail file.txt     # Last 10 lines
tail -f log.txt   # Follow growing file
less file.txt     # Paginated viewer
```

</div>

<div>

### Text Processing
```bash
grep pattern file     # Search for pattern
grep -r pattern dir/  # Recursive search
wc file.txt          # Word count
wc -l file.txt       # Line count
sort file.txt        # Sort lines
uniq file.txt        # Remove duplicates
```

</div>

</div>

---

## Redirection and Pipes - CSH

### CSH Redirection (Default Shell)
```csh
command > file.txt       # Redirect stdout to file
command >> file.txt      # Append stdout to file
command >& error.log     # Redirect both stdout and stderr
command >>& all.log      # Append both stdout and stderr
command < input.txt      # Read from file
```

### CSH Pipes
```csh
cat file.txt | grep pattern | sort | uniq
ls -la | grep "\.py$" | wc -l
ps aux | grep python | head -5
```

---

## Redirection and Pipes - BASH

### BASH Redirection
```bash
command > file.txt       # Redirect stdout to file
command >> file.txt      # Append stdout to file
command 2> error.log     # Redirect stderr
command &> all.log       # Redirect both stdout and stderr
command 2>&1             # Redirect stderr to stdout
command < input.txt      # Read from file
```

### BASH Pipes (Same as CSH)
```bash
cat file.txt | grep pattern | sort | uniq
ls -la | grep "\.py$" | wc -l
ps aux | grep python | head -5
```

---

## Environment Variables

### What Makes Them Special?
Unlike regular programming variables:
- **Inherited by child processes** (programs you run)
- **Available system-wide** within your session
- **Used by system and applications** for configuration

### Common Environment Variables
```bash
echo $HOME        # Home directory
echo $PATH        # Executable search path
echo $USER        # Current username
echo $PWD         # Current directory
echo $SHELL       # Current shell
```

---

## Environment Variables - CSH

### Setting Variables (CSH Default)
```csh
setenv MYVAR "hello world"          # Set environment variable
setenv PATH ${PATH}:/new/path       # Modify PATH

# Regular variables (not inherited)
set localvar = "local only"         # Not exported to children
```

### Unsetting Variables
```csh
unsetenv MYVAR                      # Remove environment variable
unset localvar                      # Remove regular variable
```

---

## Environment Variables - BASH

### Setting Variables (BASH)
```bash
export MYVAR="hello world"          # Set environment variable
export PATH=$PATH:/new/path         # Modify PATH

# Two-step approach
MYVAR="hello world"                 # Regular variable first
export MYVAR                        # Then export it

# Regular variables (not inherited)
localvar="local only"               # Not exported to children
```

### Unsetting Variables
```bash
unset MYVAR                         # Remove variable (env or regular)
```

---

## Using Environment Variables in Scripts

### CSH Script Example
```csh
#!/bin/csh
setenv DATA_DIR "/scratch/$USER/data"
setenv OUTPUT_DIR "$DATA_DIR/results"
echo "Processing data in $DATA_DIR"
```

### BASH Script Example
```bash
#!/bin/bash
export DATA_DIR="/scratch/$USER/data"
export OUTPUT_DIR="$DATA_DIR/results"
echo "Processing data in $DATA_DIR"
```

---

## Process Management

### Viewing Processes
```bash
ps aux              # All running processes
ps aux | grep python # Find Python processes
top                 # Dynamic process viewer
htop               # Enhanced top (if available)
```

### Job Control
```bash
command &           # Run in background
jobs               # List background jobs
fg %1              # Bring job 1 to foreground
bg %1              # Send job 1 to background
ctrl+z             # Suspend current job
ctrl+c             # Kill current job
kill PID           # Kill process by ID
```

---

## Shortcuts and Wildcards

### Tab Completion
- Press `Tab` to auto-complete filenames and commands
- Press `Tab` twice to see all possible completions

### Useful Shortcuts
```bash
ctrl+c    # Kill current command
ctrl+z    # Suspend current command
ctrl+d    # Exit shell/logout
ctrl+l    # Clear screen
ctrl+a    # Move to beginning of line
ctrl+e    # Move to end of line
```

### Wildcards
```bash
*.txt           # All .txt files
data_*.csv      # Files starting with data_
file?.txt       # file1.txt, fileA.txt, etc.
[abc]*.py       # Files starting with a, b, or c
```

---

## Text Editors

### Vi/Vim Basics
```bash
vi filename        # Open file in vi
vim filename       # Open file in vim (enhanced vi)
```

**Basic Vi Commands:**
- `i`: Insert mode
- `Esc`: Command mode
- `:w`: Save
- `:q`: Quit
- `:wq`: Save and quit
- `:q!`: Quit without saving
- `/pattern`: Search
- `dd`: Delete line

### Alternative Editors
- `nano`: User-friendly editor
- `emacs`: Powerful editor with many features

---

## Here Documents

### Creating Multi-line Input
```bash
cat << EOF > config.txt
server=hpc-cluster.edu
username=myuser
timeout=300
EOF
```

### In Scripts
```bash
#!/bin/bash
mysql -u user -p << EOF
USE database;
SELECT * FROM table;
EOF
```

---

## Regular Expressions

### Basic Patterns
```bash
grep "pattern" file.txt           # Literal match
grep "^start" file.txt           # Lines starting with "start"
grep "end$" file.txt             # Lines ending with "end"
grep "t.st" file.txt             # . matches any character
grep "colou?r" file.txt          # ? makes preceding char optional
grep "colou*r" file.txt          # * matches zero or more
grep "test[123]" file.txt        # Character class
```

### Extended Regex (with -E)
```bash
grep -E "test|exam" file.txt     # OR operator
grep -E "test+" file.txt         # + matches one or more
grep -E "(test){2}" file.txt     # Grouping and repetition
```

---

<!-- _class: lead -->

# 4. Python on HPC

---

## Python Basics Review

### Variables and Data Types
```python
# Numbers
x = 42
pi = 3.14159
result = x * pi

# Strings
name = "HPC User"
message = f"Hello, {name}!"

# Lists
data = [1, 2, 3, 4, 5]
files = ["input1.txt", "input2.txt", "input3.txt"]

# Dictionaries
config = {
    "nodes": 4,
    "cores_per_node": 24,
    "memory": "64GB"
}
```

---

## Control Flow

### Conditionals
```python
if cores > 16:
    print("High-performance node")
elif cores > 8:
    print("Medium-performance node")
else:
    print("Standard node")

# Memory check
memory_gb = 128
partition = "bigmem" if memory_gb > 64 else "compute"
```

### Loops
```python
# For loops
for i in range(10):
    print(f"Processing iteration {i}")

for filename in ["data1.csv", "data2.csv", "data3.csv"]:
    process_file(filename)

# While loops
count = 0
while count < max_iterations and not converged:
    result = run_simulation()
    count += 1
```

---

## Functions

### Basic Functions
```python
def process_data(input_file, output_file):
    """Process data from input file and save to output file."""
    data = load_data(input_file)
    processed = analyze(data)
    save_results(processed, output_file)
    return processed

def calculate_memory_needed(data_size_gb, overhead_factor=1.5):
    """Calculate memory requirements with overhead."""
    return int(data_size_gb * overhead_factor)

# Usage
memory_req = calculate_memory_needed(32)  # 48 GB
```

### Advanced Function Features
```python
def parallel_process(*files, workers=4, **kwargs):
    """Process multiple files in parallel."""
    for file in files:
        submit_job(file, workers=workers, **kwargs)

# Usage
parallel_process("data1.csv", "data2.csv", workers=8, memory="16GB")
```

---

## Python on HPC: Best Practices

### Module Loading
```bash
# In SLURM script or interactive session
module load python/3.9
module load python/3.9-conda  # Or conda-based Python
```

### Virtual Environments
```bash
# Create virtual environment
python -m venv myproject_env

# Activate
source myproject_env/bin/activate

# Install packages
pip install numpy pandas scikit-learn

# Deactivate
deactivate
```

---

## Python HPC Libraries

### Essential Scientific Libraries
```python
import numpy as np          # Numerical computing
import pandas as pd         # Data manipulation
import matplotlib.pyplot as plt  # Plotting
import scipy                # Scientific computing
import scikit-learn        # Machine learning
```

### Parallel Processing
```python
import multiprocessing as mp
import concurrent.futures
from joblib import Parallel, delayed

# Multiprocessing example
def process_chunk(data_chunk):
    return analyze(data_chunk)

with mp.Pool(processes=8) as pool:
    results = pool.map(process_chunk, data_chunks)
```

---

## Sample Python HPC Workflow

```python
#!/usr/bin/env python3
"""
HPC Data Processing Pipeline
"""
import sys
import os
import numpy as np
import pandas as pd
from multiprocessing import Pool

def process_file(filename):
    """Process a single data file."""
    print(f"Processing {filename}")
    data = pd.read_csv(filename)
    # Your analysis here
    result = data.groupby('category').mean()
    output_name = f"processed_{os.path.basename(filename)}"
    result.to_csv(output_name)
    return output_name

def main():
    # Get input files from command line
    input_files = sys.argv[1:]
    
    # Determine number of processes from SLURM
    n_processes = int(os.environ.get('SLURM_CPUS_PER_TASK', 1))
    
    # Process files in parallel
    with Pool(processes=n_processes) as pool:
        results = pool.map(process_file, input_files)
    
    print(f"Processed {len(results)} files")

if __name__ == "__main__":
    main()
```

---

<!-- _class: lead -->

# 5. SLURM Job Management

---

## What is SLURM?

**Simple Linux Utility for Resource Management**

- Job scheduler and resource manager
- Allocates compute resources to users
- Manages job queues and priorities
- Monitors resource usage

### Key Concepts
- **Jobs**: Your computational tasks
- **Partitions**: Groups of compute nodes
- **Queues**: Where jobs wait for resources

---

## Job Submission

### Interactive Jobs
```bash
# Request interactive session
srun --pty --nodes=1 --time=01:00:00 bash

# With specific resources
srun --pty --nodes=1 --cpus-per-task=4 --mem=8G --time=02:00:00 bash
```

### Batch Jobs
```bash
# Submit batch script
sbatch myjob.slurm

# Submit with parameters
sbatch --nodes=2 --time=04:00:00 myscript.sh
```

---

## Sample SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=my-analysis
#SBATCH --output=output_%j.txt
#SBATCH --error=error_%j.txt
#SBATCH --time=02:00:00
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=16G
#SBATCH --partition=compute

module load python/3.9
python my_analysis.py
```

---

## Job Management Commands

### Monitoring Jobs
```bash
squeue                    # View all jobs
squeue -u $USER          # View your jobs
scontrol show job JOBID  # Detailed job info
```

### Managing Jobs
```bash
scancel JOBID            # Cancel specific job
scancel -u $USER         # Cancel all your jobs
```

### System Information
```bash
sinfo                    # View partitions/nodes
scontrol show partition  # Partition details
```

---

## Job Priority and Queues

### Factors Affecting Priority
- **Wait time**: Longer wait = higher priority
- **Fair share**: Based on recent usage
- **Job size**: Larger jobs may have different priority
- **Partition**: Different queues have different priorities

### Common Partitions
- `debug`: Short jobs, quick turnaround
- `compute`: General purpose computing
- `gpu`: GPU-enabled nodes
- `bigmem`: High-memory nodes
- `long`: Extended runtime jobs

---

<!-- _class: lead -->

# 6. Best Practices & Next Steps

---

## HPC Best Practices

### Resource Management
- **Request appropriate resources** (don't over-allocate)
- **Use scratch storage** for temporary files
- **Clean up after jobs** complete
- **Monitor job efficiency** with `seff JOBID`

### Data Management
- **Backup important data** regularly
- **Use compression** for large datasets
- **Organize projects** in clear directory structures
- **Document your workflow** and dependencies

---

## Workflow Example

```bash
# 1. Prepare environment
module load python/3.9
source my_project_env/bin/activate

# 2. Organize data
mkdir -p /scratch/$USER/project/{data,results,logs}

# 3. Submit job
sbatch --array=1-100 process_array.slurm

# 4. Monitor progress
watch squeue -u $USER

# 5. Collect results
cd /scratch/$USER/project/results
tar -czf results_$(date +%Y%m%d).tar.gz *.csv
```

---

## Common Pitfalls to Avoid

### Resource Issues
- ❌ Running on login nodes for compute tasks
- ❌ Requesting too much memory/time "just in case"
- ❌ Not using parallel processing when beneficial

### File Management
- ❌ Storing large files in home directory
- ❌ Not cleaning up temporary files
- ❌ Hard-coding absolute paths in scripts

### Job Management
- ❌ Not testing code before submitting large jobs
- ❌ Submitting many small jobs instead of job arrays
- ❌ Not monitoring job efficiency

---

## Getting Help

### Documentation
- HPC center documentation and tutorials
- `man command` for detailed command help
- SLURM documentation online

### Support Resources
- HPC help desk/support tickets
- User forums and communities
- Office hours or training sessions

### Useful Commands for Troubleshooting
```bash
scontrol show job JOBID    # Detailed job information
sacct -j JOBID             # Job accounting information  
seff JOBID                 # Job efficiency report
module avail               # Available software modules
quota -u $USER             # Check disk usage limits
```

---

## Next Steps

### Intermediate Topics
- **Job arrays** for parameter sweeps
- **GPU computing** with CUDA/OpenCL
- **Container** usage (Singularity/Docker)
- **Workflow managers** (Snakemake, Nextflow)

### Advanced Topics  
- **MPI** for distributed computing
- **Performance profiling** and optimization
- **Custom software** compilation and installation
- **Advanced scheduling** and resource allocation

### Continue Learning
- Attend HPC workshops and training
- Join user communities
- Practice with real projects
- Experiment with new tools and techniques

---

<!-- _class: lead -->

# Questions?

## Resources
- **HPC Documentation**: [Your institution's HPC docs]
- **SLURM Documentation**: https://slurm.schedmd.com/
- **Python Scientific Stack**: https://scipy.org/
- **Linux/Unix Tutorials**: Various online resources

**Thank you for attending!**

---
