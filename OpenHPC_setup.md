## Install OpenHPC on Single Node (i.e. on PackwoodGPU computer)

This article indicates how to install OpenHPC on a single node running Rocky 9.3, using Warewulf and Slurm.

This method is based on that given in [manbaritone/OpenHPC-Installation/09_OpenHPC Slurm Setup for Single Node.md](https://github.com/manbaritone/OpenHPC-Installation/blob/master/09_OpenHPC%20Slurm%20Setup%20for%20Single%20Node.md)

**Master and Compute**
- Hostname: nodename (change this to what it should be. Talk to Daniel PAckwood or someone in the group for this name.)
- eno2: public network XX.XXX.XX.XXX (depend on your ip address)

## Setting up the PackwoodGPU Computer

1. Change the hostname for the node, and update the ``/etc/hosts`` file.

```bash
sudo hostnamectl set-hostname nodename

sudo vim /etc/hosts

# Add this line to /etc/hosts file --> XX.XXX.XX.XXX nodename nodename.local
```

2. Disable firewall.

```bash
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```

3. Disable SELINUX.

```bash
sudo vim /etc/selinux/config

# Change SELINUX line to --> SELINUX=disabled
```

4. Reboot the computer.

```bash
sudo reboot
```

5. Update Rocky.

```bash
sudo yum -y update
```

## Install OpenHPC and Slurm Setup


1. Install the OpenHPC repository on the PackwoodGPU computer.

```bash
sudo yum -y install http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/ohpc-release-3-1.el9.x86_64.rpm
```

2. Install the basic OpenHPC packages on the PackwoodGPU computer.

```bash
sudo yum -y install ohpc-base ohpc-warewulf
```

3. Install slurm.

```bash
sudo yum -y install ohpc-slurm-server
sudo yum -y install ohpc-slurm-client
sudo cp /etc/slurm/slurm.conf.ohpc /etc/slurm/slurm.conf
```

4. Restart and enable services for running slurm.

```bash
sudo systemctl enable mariadb.service 
sudo systemctl restart mariadb
sudo systemctl enable httpd.service
sudo systemctl restart httpd
sudo systemctl enable dhcpd.service
```

5. Install OpenHPC for running the "compute node" on the PackwoodGPU computer.

```bash
sudo yum -y install ohpc-base-compute
```

6. Install the ``modules`` user enviroment for the compute and master node, both on the PackwoodGPU computer.

```bash
sudo yum -y install lmod-ohpc
```

7. Create basic values for OpenHPC.

```bash
sudo wwinit database 
sudo wwinit ssh_keys
```

8. Update the slurm configurations for the PackwoodGPU computer.

    * Copy the following and copy it into the ``/etc/slurm/slurm.conf`` file. If you want to make adjustments to it, you can adjust by using configuration tool at https://slurm.schedmd.com/configurator.html

```bash title="/etc/slurm/slurm.conf"
--8<-- "Files/slurm.conf"
```

9. Add the ``gres.conf`` file for GPU allocation.

    * Please check number of GPU, e.g. nvidia[0-1] is 2 GPUs

```bash title="/etc/slurm/gres.conf"
--8<-- "Files/gres.conf"
```

10. Restart the Slurm and Munge services.

```bash
sudo systemctl enable munge
sudo systemctl enable slurmctld
sudo systemctl enable slurmd
sudo systemctl start munge
sudo systemctl start slurmctld
sudo systemctl start slurmd
```

11. Determine memlock values

```bash
sudo perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' /etc/security/limits.conf
sudo perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' /etc/security/limits.conf
sudo perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' $CHROOT/etc/security/limits.conf 
sudo perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' $CHROOT/etc/security/limits.conf
```

12. Test resource by checking that the command below give back information about the PackwoodGPU computer.

```bash 
sudo scontrol show nodes
```

## Installation essential modules/softwares


1.  Install the Intel Complier (Intel oneAPI Base)

    * Ref: https://software.intel.com/content/www/us/en/develop/tools/oneapi/base-toolkit/download.html?operatingsystem=linux&distributions=yumpackagemanager

```bash
sudo bash -c 'cat << EOF > /etc/yum.repos.d/oneAPI.repo
[oneAPI]
name=Intel(R) oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
EOF'
```
```bash
sudo yum upgrade intel-basekit
```

2. Install OHPC basic additional packages (you can check additional packages from http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/).

```bash
sudo yum install -y EasyBuild-ohpc autoconf-ohpc automake-ohpc cmake-ohpc gnu12-compilers-ohpc hwloc-ohpc libtool-ohpc python3-Cython-ohpc spack-ohpc valgrind-ohpc
```

3. Install the ``GNU12`` compiler.

```bash
sudo yum -y install gnu12-compilers-ohpc
```

4. Install MPI Stacks for parallelisation.

```bash
sudo yum -y install openmpi4-pmix-gnu12-ohpc mpich-ofi-gnu12-ohpc mpich-ucx-gnu12-ohpc
```

5. Install performance tools for aiding in application performance analysis.

```bash
sudo yum -y install ohpc-gnu12-perf-tools
```

6. Setup the default development environment that configures the default environment to enables autotools, the GNU compiler toolchain, and the OpenMPI stack.

```bash
sudo yum -y install lmod-defaults-gnu12-openmpi4-ohpc
```

7. Install useful 3rd party libraries and tools

```bash
# Install 3rd party libraries/tools meta-packages built with GNU toolchain
sudo yum -y install ohpc-gnu12-serial-libs ohpc-gnu12-io-libs ohpc-gnu12-python-libs ohpc-gnu12-runtimes

# Install parallel lib meta-packages for all available MPI toolchains
sudo yum -y install ohpc-gnu12-mpich-parallel-libs ohpc-gnu12-openmpi4-parallel-libs
```

8. Install other 3rd party development libraries and tools

    * Some of these may not install due to changes with Intel. This is ok, just install whatever you can.

```bash
# Enable Intel oneAPI and install OpenHPC compatibility packages
sudo yum -y install intel-oneapi-toolkit-release-ohpc
sudo rpm --import https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
sudo yum -y install intel-compilers-devel-ohpc
sudo yum -y install intel-mpi-devel-ohpc

# Install 3rd party libraries/tools meta-packages built with Intel toolchain
sudo yum -y install openmpi4-pmix-intel-ohpc
sudo yum -y install ohpc-intel-serial-libs
sudo yum -y install ohpc-intel-geopm
sudo yum -y install ohpc-intel-io-libs
sudo yum -y install ohpc-intel-perf-tools
sudo yum -y install ohpc-intel-python3-libs
sudo yum -y install ohpc-intel-mpich-parallel-libs
sudo yum -y install ohpc-intel-mvapich2-parallel-libs
sudo yum -y install ohpc-intel-openmpi4-parallel-libs
sudo yum -y install ohpc-intel-impi-parallel-libs
```

<!-- 

3. Install the OpenHPC GNU9 basic additional packages (you can check additional packages from http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/).

```bash
sudo yum install -y ohpc-gnu9* R-gnu9-ohpc adios-gnu9* boost-gnu9* fftw-gnu9* hdf5-gnu9* mpich-ofi-gnu9* mpich-ucx-gnu9* mvapich2-gnu9* netcdf-gnu9* openmpi4-gnu9* pdtoolkit-gnu9* phdf5-gnu9* pnetcdf-gnu9* python3-mpi4py-gnu9* sionlib-gnu9* 
```

4. Install the OpenHPC Intel basic additional packages (you can check additional packages from http://repos.openhpc.community/OpenHPC/3/EL_9/x86_64/).

```bash
sudo yum install -y ohpc-intel* intel-compilers-devel-ohpc intel-mpi-devel-ohpc adios-intel* boost-intel* hdf5-intel* mpich-ofi-intel* mpich-ucx-intel* mvapich2-intel* netcdf-intel* openmpi4-intel* pdtoolkit-intel* phdf5-intel* pnetcdf-intel* python3-mpi4py-intel* sionlib-intel* 
``` -->


## Restart the Resource Manager

It is now a good idea to restart the resource manager and check that it is running

```bash
sudo systemctl restart munge
sudo systemctl restart slurmctld
sudo systemctl restart slurmd
sudo systemctl status munge
sudo systemctl status slurmctld
sudo systemctl status slurmd
```




