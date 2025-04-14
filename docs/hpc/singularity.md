# Setting up Singularity/Apptainer on OpenHPC with Rocky Linux

This guide provides instructions for setting up Singularity (also known as Apptainer after the project fork) on an OpenHPC cluster running Rocky Linux.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Configuration for OpenHPC](#configuration-for-openhpc)
- [Using Singularity in OpenHPC Environment](#using-singularity-in-openhpc-environment)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before beginning the installation, ensure you have:
- Administrator access (sudo privileges) on your OpenHPC cluster
- Internet access to download packages
- Rocky Linux (8.x or 9.x) with OpenHPC installed

## Installation Steps

### 1. Install Required Dependencies

```bash
# Install EPEL repository
sudo dnf install -y epel-release

# Install required packages
sudo dnf install -y wget git gcc gcc-c++ make \
    libseccomp-devel squashfs-tools cryptsetup \
    rpm-build autoconf automake libtool
```

### 2. Install Go Programming Language

Singularity/Apptainer is written in Go and requires Go to build from source.

```bash
export VERSION=1.19 OS=linux ARCH=amd64
wget https://dl.google.com/go/go$VERSION.$OS-$ARCH.tar.gz
sudo tar -C /usr/local -xzvf go$VERSION.$OS-$ARCH.tar.gz
rm go$VERSION.$OS-$ARCH.tar.gz
```

### 3. Set Up Go Environment Variables

```bash
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=${HOME}/go' >> ~/.bashrc
source ~/.bashrc
```

Verify Go installation:
```bash
go version
```

### 4. Download and Install Singularity/Apptainer

You can choose to install either Singularity or Apptainer:

#### Option A: Install Singularity

```bash
export VERSION=3.8.7  # adjust to desired version
wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz
tar -xzf singularity-ce-${VERSION}.tar.gz
cd singularity-ce-${VERSION}
```

#### Option B: Install Apptainer (recommended for newer environments)

```bash
export VERSION=1.1.9  # adjust to desired version
wget https://github.com/apptainer/apptainer/releases/download/v${VERSION}/apptainer-${VERSION}.tar.gz
tar -xzf apptainer-${VERSION}.tar.gz
cd apptainer-${VERSION}
```

### 5. Configure and Compile

```bash
./mconfig
cd builddir
make
sudo make install
```

### 6. Verify Installation

For Singularity:
```bash
singularity --version
```

For Apptainer:
```bash
apptainer --version
```

## Configuration for OpenHPC

### Basic Configuration

1. Create system-wide configuration directory:
```bash
sudo mkdir -p /etc/singularity
```

2. Copy default configuration file:
```bash
# For Singularity
sudo cp /usr/local/etc/singularity/singularity.conf /etc/singularity/

# For Apptainer
sudo cp /usr/local/etc/apptainer/apptainer.conf /etc/apptainer/
```

### MPI Integration

To ensure proper integration with OpenHPC's MPI libraries:

```bash
# For Singularity
sudo sed -i 's/bind path = \/etc\/localtime/bind path = \/etc\/localtime\nbind path = \/opt\/ohpc/' /etc/singularity/singularity.conf

# For Apptainer
sudo sed -i 's/bind path = \/etc\/localtime/bind path = \/etc\/localtime\nbind path = \/opt\/ohpc/' /etc/apptainer/apptainer.conf
```

### Module File Creation (Optional)

Create a module file to make Singularity/Apptainer available to users:

```bash
sudo mkdir -p /opt/ohpc/pub/modulefiles/singularity
```

Create a modulefile for Singularity/Apptainer:

```bash
cat << 'EOF' | sudo tee /opt/ohpc/pub/modulefiles/singularity/3.8.7
#%Module1.0#####################################################################
proc ModulesHelp { } {
    puts stderr "This module loads the Singularity container runtime"
    puts stderr "\nVersion 3.8.7\n"
}

module-whatis "Name: Singularity container runtime"
module-whatis "Version: 3.8.7"
module-whatis "Category: system tool"
module-whatis "Description: Application and environment virtualization"
module-whatis "URL: https://sylabs.io/singularity/"

prepend-path    PATH            /usr/local/bin
prepend-path    MANPATH         /usr/local/share/man
EOF
```

## Using Singularity in OpenHPC Environment

### Basic Usage

1. Pull an example container:
```bash
singularity pull library://sylabsed/examples/lolcow
```

2. Run the container:
```bash
singularity run lolcow_latest.sif
```

3. Shell into a container:
```bash
singularity shell lolcow_latest.sif
```

### Integration with SLURM

Create a simple batch job script for SLURM:

```bash
cat > singularity_job.sh << 'EOF'
#!/bin/bash
#SBATCH -J singularity_test
#SBATCH -o singularity_test.%j.out
#SBATCH -e singularity_test.%j.err
#SBATCH -p compute
#SBATCH -N 1
#SBATCH -n 1

# Load singularity module if it exists
module load singularity 2>/dev/null || true

# Run container
singularity run /path/to/your/container.sif
EOF
```

Submit the job:
```bash
sbatch singularity_job.sh
```

### MPI Example with OpenHPC

Create an MPI-enabled job script:

```bash
cat > mpi_singularity_job.sh << 'EOF'
#!/bin/bash
#SBATCH -J mpi_singularity
#SBATCH -o mpi_singularity.%j.out
#SBATCH -e mpi_singularity.%j.err
#SBATCH -p compute
#SBATCH -N 2
#SBATCH -n 16

# Load modules
module load openmpi
module load singularity

# Run MPI job
mpirun -n $SLURM_NTASKS singularity exec /path/to/mpi-container.sif /path/to/mpi/program
EOF
```

## Advanced Configuration

### Enable User Namespaces (if needed)

If you want to allow unprivileged users to build containers, you may need to enable user namespaces:

```bash
echo 'user.max_user_namespaces=15000' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Configure Bind Mounts

Edit the configuration file to add additional bind mounts:

```bash
sudo sed -i '/^bind path = /a bind path = /scratch,/data,/projects' /etc/singularity/singularity.conf
```

### Setup Registry Authentication

Create a remote configuration for registry access:

```bash
singularity remote login SylabsCloud
```

## Troubleshooting

### Common Issues

1. **"FATAL: container creation failed: mount ..."**:
   - Check if kernel supports overlayfs
   - Ensure user namespaces are enabled

2. **"FATAL: container creation failed: while creating overlay mount"**:
   ```bash
   sudo singularity config fakeroot --add username
   ```

3. **MPI jobs failing**:
   - Ensure proper bind mounting of OpenHPC directories
   - Check that MPI implementations inside and outside the container are compatible

### Diagnostic Commands

Check system capabilities:
```bash
singularity capability list
```

Verify configuration:
```bash
singularity config global
```

Check for libseccomp version:
```bash
rpm -q libseccomp
```

---

For additional support, refer to the official documentation:
- [Singularity Documentation](https://sylabs.io/docs/)
- [Apptainer Documentation](https://apptainer.org/docs/)
- [OpenHPC Documentation](https://openhpc.community/documentation/)