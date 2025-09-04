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

## File Comparison

### Comparing Files
```bash
diff file1.txt file2.txt           # Show differences between files
diff -u file1.txt file2.txt        # Unified diff format
diff -r dir1/ dir2/                # Compare directories recursively
```

### HPC Use Cases
```bash
# Compare configuration files
diff config.old config.new

# Check job script changes  
diff -u job_v1.slurm job_v2.slurm

# Verify backup integrity
diff -r /home/user/data /backup/data
```

---

## Archive & Compression

### Creating Archives
```bash
tar -cf archive.tar files/         # Create tar archive
tar -czf archive.tar.gz files/     # Create compressed tar (gzip)
tar -cjf archive.tar.bz2 files/    # Create compressed tar (bzip2)
zip -r archive.zip files/          # Create zip archive
```

### Extracting Archives
```bash
tar -xf archive.tar                # Extract tar archive
tar -xzf archive.tar.gz            # Extract gzipped tar
tar -xjf archive.tar.bz2           # Extract bzip2 tar
unzip archive.zip                  # Extract zip archive
```

---

## Archive & Compression - HPC Workflows

### Practical Examples
```bash
# Archive results for transfer and list contents
tar -czf results_$(date +%Y%m%d).tar.gz output/
tar -tzf results_$(date +%Y%m%d).tar.gz    # List files in archive

# Compress large datasets
gzip large_dataset.csv             # Creates large_dataset.csv.gz
gunzip large_dataset.csv.gz        # Uncompress
```

### Best Practices
- **Use compression** for large files and transfers
- **Include timestamps** in archive names
- **Test extraction** before deleting originals

---

## Redirection and Pipes - CSH

### CSH Redirection (Default Shell)
```bash
command > file.txt       # Redirect stdout to file
command >> file.txt      # Append stdout to file
command >& error.log     # Redirect both stdout and stderr
command >>& all.log      # Append both stdout and stderr
command < input.txt      # Read from file
```

### CSH Pipes
```bash
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
```bash
setenv MYVAR "hello world"          # Set environment variable
setenv PATH ${PATH}:/new/path       # Modify PATH

# Regular variables (not inherited)
set localvar = "local only"         # Not exported to children
```

### Unsetting Variables
```bash
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
```bash
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

## Environment Modules (Lmod)

### What are Environment Modules?
- **Software management system** for HPC environments
- **Dynamically modify environment** (PATH, LD_LIBRARY_PATH, etc.)
- **Multiple versions** of software can coexist
- **Clean environment** - load only what you need

### Why Use Modules?
- **Avoid conflicts** between different software versions
- **Reproducible environments** for research
- **Easy switching** between software stacks
- **Optimized builds** for HPC hardware

---

## Module Commands

### Discovering Available Software
```bash
module avail                    # Show all available modules
module avail python            # Show Python modules
module avail 2>&1 | grep -i gcc  # Search for GCC modules
```

### Managing Loaded Modules
```bash
module list                    # Show currently loaded modules
module load python/3.9        # Load specific Python version
module load gcc/11.2.0         # Load GCC compiler
module unload python           # Unload Python module
module purge                   # Unload all modules
```

---

## Module Commands (continued)

### Getting Information
```bash
module show python/3.9         # Show module details
module help python/3.9         # Show module help
module whatis python/3.9       # Brief description
```

### Practical Tips
- **Load order matters** - some modules depend on others
- **Check conflicts** - some modules cannot be loaded together
- **Use specific versions** for reproducibility
- **Start fresh** with `module purge` when troubleshooting

---

## Module Workflow Example

### Typical Module Loading Sequence
```bash
# Start with clean environment
module purge                   # Remove all loaded modules

# Load dependencies first
module load gcc/11.2.0         # Load compiler first
module load openmpi/4.1.0      # Load MPI library
module load python/3.9         # Then load Python

# Verify what's loaded
module list                    # Check loaded modules

# Load additional tools as needed
module load git/2.35.0         # Version control
module load cmake/3.22.0       # Build system
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

## Shortcuts and Tab Completion

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

---

## Wildcards

### Pattern Matching
```bash
*.txt           # All .txt files
data_*.csv      # Files starting with data_
file?.txt       # file1.txt, fileA.txt, etc.
[abc]*.py       # Files starting with a, b, or c
[0-9]*.log      # Files starting with any digit
*.[ch]          # Files ending in .c or .h
```

---

## Text Editors - Vi/Vim

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

---

## Text Editors - Alternatives

### User-Friendly Options
- **nano**: Simple, intuitive editor with on-screen help
- **emacs**: Powerful editor with extensive features
- **gedit**: GUI text editor (if X11 forwarding enabled)

### Why Use Alternatives?
- **nano**: Great for beginners, shows key commands at bottom
- **emacs**: Advanced features, extensible
- **GUI editors**: Familiar interface for those new to command line

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

## Regular Expressions - Introduction

### What are Regular Expressions?
- **Pattern matching language** for searching text
- **Powerful tool** for finding, filtering, and extracting data
- **Widely used** in text processing, log analysis, data cleaning
- **Essential skill** for HPC data management

### Common Use Cases
- **Log file analysis**: Find error patterns
- **Data validation**: Check file formats
- **File filtering**: Select files by naming patterns
- **Text extraction**: Pull specific information from output

---

## Regular Expressions - Basic Patterns

### Literal and Position Matching
```bash
grep "error" logfile.txt         # Literal match
grep "^ERROR" logfile.txt        # Lines starting with "ERROR"
grep "completed$" logfile.txt    # Lines ending with "completed"
```

### Character Classes and Ranges
```bash
grep "t.st" file.txt             # . matches any single character
grep "test[123]" file.txt        # Character class [123]
grep "test[0-9]" file.txt        # Character range [0-9]
```

### Quantifiers
```bash
grep "colou*r" file.txt          # * matches zero or more
grep "colou.*r" file.txt         # .* matches any characters
```

---

## Regular Expressions - Extended Patterns

### Extended Regex (Requires -E)
```bash
grep -E "test|exam" file.txt     # OR operator |
grep -E "test+" file.txt         # + matches one or more
grep -E "test?" file.txt         # ? makes preceding char optional
grep -E "test{2}" file.txt       # {2} matches exactly 2
grep -E "test{2,5}" file.txt     # {2,5} matches 2 to 5
```

### Advanced Quantifiers
```bash
grep -E "(error|warning):" log.txt   # Alternation with grouping
grep -E "file[0-9]+\.txt" file.txt   # One or more digits
grep -E "[a-zA-Z]+@[a-zA-Z]+" file.txt # Email-like patterns
```

---

## Regular Expressions - Grouping and Capture

### Using Parentheses for Grouping
```bash
grep -E "(test|exam)ing" file.txt    # Grouping with alternation
grep -E "file(1|2|3).txt" file.txt   # Multiple options
grep -E "([0-9]+)" file.txt          # Group numbers
grep -E "([a-zA-Z]+)\.txt" file.txt  # Group filenames
```

### Practical Examples with Capture Groups
```bash
# Extract job IDs from SLURM output
grep -E "Job ([0-9]+)" slurm.out

# Find email patterns
grep -E "([a-zA-Z0-9]+)@([a-zA-Z0-9]+)" file.txt

# Match IP addresses
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" log.txt
```

### Note on Capture Groups
- **Grouping**: Parentheses group patterns for operators
- **Extraction**: Some tools can extract grouped content
- **grep**: Shows matches but doesn't extract groups separately

---

## Text Processing with sed

### What is sed?
- **Stream Editor** for filtering and transforming text
- **Non-interactive** - works in pipelines and scripts
- **Pattern-based** editing with regular expressions
- **Essential tool** for HPC log processing and data cleaning

---

## Basic sed Operations

### Text Substitution
```bash
# Substitute (replace) text
sed 's/old/new/' file.txt          # Replace first occurrence per line
sed 's/old/new/g' file.txt         # Replace all occurrences
sed 's/ERROR/WARNING/g' log.txt     # Change log levels
```

### Deleting Lines
```bash
# Delete lines
sed '/pattern/d' file.txt           # Delete lines containing pattern
sed '1,5d' file.txt                 # Delete lines 1-5
sed '/^$/d' file.txt                # Delete empty lines
```

---

## Advanced sed Examples

### HPC-Specific sed Usage
```bash
# Extract job information from SLURM logs
sed -n '/Job [0-9]/p' slurm.out     # Print lines with job numbers
sed 's/.*Job \([0-9]*\).*/\1/' log.txt  # Extract job IDs

# Clean up data files
sed 's/,/ /g' data.csv              # Replace commas with spaces
sed 's/^[ \t]*//' file.txt          # Remove leading whitespace
sed '/^#/d' config.txt              # Remove comment lines

# Modify configuration files
sed 's/nodes=4/nodes=8/' job.sh     # Update node count
sed -i 's/old/new/g' file.txt       # Edit file in-place
```

---

## Text Processing with awk

### What is awk?
- **Pattern-scanning and processing language**
- **Field-oriented** - automatically splits lines into fields
- **Built-in variables** like NF (number of fields), NR (record number)
- **Powerful for** data extraction and reporting

### Basic awk Structure
```bash
# Basic syntax: awk 'pattern { action }' file
awk '{ print }' file.txt            # Print all lines (like cat)
awk '{ print $1 }' file.txt         # Print first field
awk '{ print $1, $3 }' file.txt     # Print first and third fields
awk '{ print NF }' file.txt         # Print number of fields per line
awk '{ print NR, $0 }' file.txt     # Print line numbers
```

---

## Advanced awk Examples

### HPC Data Processing
```bash
# Process SLURM output
awk '/COMPLETED/ { print $1, $6 }' squeue.out  # Job ID and state
awk '$3 > 1000 { print $1 }' memory.log        # Jobs using >1000MB

# Calculate statistics
awk '{ sum += $2 } END { print sum }' data.txt # Sum second column
awk '{ sum += $1; count++ } END { print sum/count }' # Average
awk 'BEGIN { max = 0 } { if ($1 > max) max = $1 } END { print max }'

# Format output
awk '{ printf "%-10s %8.2f\n", $1, $2 }' data.txt  # Formatted columns
awk -F: '{ print $1, $3 }' /etc/passwd              # Custom field separator
```

### Practical Tips
- **Use -F** to specify field separators (`:`, `,`, etc.)
- **Combine with pipes** for complex processing
- **Variables persist** throughout processing

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
config = {"nodes": 4, "cores_per_node": 24, "memory": "64GB"}
```

---

## Python Conditionals

```python
# If/elif/else statements
if cores > 16:
    print("High-performance node")
elif cores > 8:
    print("Medium-performance node")
else:
    print("Standard node")

# Conditional expressions (ternary operator)
memory_gb = 128
partition = "bigmem" if memory_gb > 64 else "compute"

# Boolean conditions
is_ready = data_loaded and model_trained
should_continue = not error_occurred and iterations < max_iter
```

---

## Python Loops

### For Loops
```python
# Range-based loops
for i in range(10):
    print(f"Processing iteration {i}")

# Iterating over lists
for filename in ["data1.csv", "data2.csv", "data3.csv"]:
    process_file(filename)
```

### While Loops
```python
# Condition-based loops
count = 0
while count < max_iterations and not converged:
    result = run_simulation()
    count += 1
```

---

## Python Functions - Basic

### Function Definition and Usage
```python
def process_data(input_file, output_file):
    """Process data from input file and save to output file."""
    data = load_data(input_file)
    processed = analyze(data)
    save_results(processed, output_file)
    return processed

# Function with default parameters
def calculate_memory_needed(data_size_gb, overhead_factor=1.5):
    """Calculate memory requirements with overhead."""
    return int(data_size_gb * overhead_factor)

# Usage examples
result = process_data("input.csv", "output.csv")
memory_req = calculate_memory_needed(32)  # Uses default overhead
memory_req = calculate_memory_needed(32, 2.0)  # Custom overhead
```

---

## Python Functions - Advanced

### Variable Arguments and Keyword Arguments
```python
def parallel_process(*files, workers=4, **kwargs):
    """Process multiple files in parallel.
    
    *files: Variable number of file arguments
    **kwargs: Additional keyword arguments passed to jobs
    """
    for file in files:
        submit_job(file, workers=workers, **kwargs)

# Usage with variable arguments
parallel_process("data1.csv", "data2.csv", "data3.csv", 
                workers=8, memory="16GB", time="02:00:00")
```

---

## Python Lambda Functions

### Anonymous Functions
```python
# Lambda functions are short, inline functions
square = lambda x: x * x
add = lambda x, y: x + y

# Usage
result = square(5)  # Returns 25
sum_val = add(3, 4)  # Returns 7
```

### Practical HPC Examples
```python
# Sort files by size
files.sort(key=lambda f: os.path.getsize(f))

# Filter completed jobs
completed = filter(lambda job: job.status == "COMPLETED", jobs)
```

---

## Python on HPC: Best Practices

### Module Loading
```bash
# In SLURM script or interactive session
module load python/3.9
module load python/3.9-conda  # Or conda-based Python
```

### Why Use Environment Management?
- **Isolated dependencies** for different projects
- **Reproducible environments** across systems
- **Avoid version conflicts** between packages
- **Clean project organization**

---

## Python Virtual Environments

### Creating and Using venv
```bash
# Create virtual environment
python -m venv myproject_env

# Activate environment
source myproject_env/bin/activate

# Install packages
pip install numpy pandas scikit-learn matplotlib

# Save environment
pip freeze > requirements.txt

# Deactivate when done
deactivate
```

---

## Conda Environments

### Creating and Using Conda
```bash
# Load conda module
module load python/3.9-conda

# Create conda environment
conda create -n myproject python=3.9

# Activate environment
conda activate myproject

# Install packages (conda or pip)
conda install numpy pandas matplotlib
conda install -c conda-forge scikit-learn
pip install custom-package
```

---

## Managing Conda Environments

### Environment Management Commands
```bash
# List environments
conda env list

# Export environment
conda env export > environment.yml

# Create from file
conda env create -f environment.yml

# Remove environment
conda env remove -n myproject

# Deactivate
conda deactivate
```

---

## Conda vs Pip

### Key Differences
- **conda**: Binary packages, faster installs, dependency resolution
- **pip**: More packages available, source installs from PyPI
- **conda**: Can install non-Python dependencies (C libraries, etc.)
- **pip**: Python-only package manager

### Best Practices for HPC
- **Start with conda** for scientific packages (numpy, scipy, pandas)
- **Use pip** for packages not available in conda
- **Install conda packages first**, then pip packages
- **Avoid mixing** conda and pip for the same package

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

# Multiprocessing example with lambda
with mp.Pool(processes=8) as pool:
    results = pool.map(lambda chunk: analyze(chunk), data_chunks)
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
- `short`: Short jobs, quick turnaround
- `production`: General purpose computing
- `accel`: GPU-enabled nodes
- `whole_node`: Whole node jobs

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

---

## Troubleshooting Commands

### SLURM Job Debugging
```bash
scontrol show job JOBID    # Detailed job information
sacct -j JOBID             # Job accounting information  
seff JOBID                 # Job efficiency report
squeue -u $USER            # Check your running jobs
scancel JOBID              # Cancel problematic job
```

### System Information
```bash
module avail               # Available software modules
module list                # Currently loaded modules
quota -u $USER             # Check disk usage limits
df -h /scratch/$USER       # Check scratch space usage
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
