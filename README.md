# singularity_container
Homemade container image for scientific computing.

Continer image
- "centos7-gnu": CentOS 7.4, GNU 4.8.5 (C,C++ and Fortran), Boost, OpenBLAS, FFTW, numctl and hwloc. 
- "centos7-gnu-cuda8": image "centos7-gnu" plus cuda 8.0

How do we prepare the homemade container image?
Assume you have a linux box with singularity. To install [Singularity](http://singularity.lbl.gov/install-linux) is quite straightforward. If you have already installed [openhpc](https://openhpc.community/) on your own linux box, it is even simple. The folowing is the procedure to produce the container image.
- Change to superuser mode in your own linux box. 
- Grab the latest centos image from docker offical web site and make the distributation as the working directory for adding more packages
  ```
  singularity build --sandbox centos7-gpu/ docker://centos
  ```
- Install the packages that you would like to pack into the container
  ```
  ## Obtain the shell in container's OS and make the working directory writable.
  singularity shell --writable centos7-gpu/ 
  Singularity centos7-gpu:~> yum install gcc-c++ gcc gcc-gfortran
  Singularity centos7-gpu:~> yum install boost openblas fftw hwloc* numactl*
  ## Download the rpm of CUDA from NVIDIA web and store it to the host
  ## Install CUDA rpm in the container
  Singularity centos7-gpu:~> rpm -Uvh cuda-repo-rhel7-9.0.176-1.x86_64.rpm
  Singularity centos7-gpu:~> yum install epel-release
  Singularity centos7-gpu:~> yum install cuda-9-0.x86_64
  ```
- Convert the working directory into a image
  ```
  singularity build centos7-gnu-cuda9.img centos7-gpu/
  ```
- Copy and test the image to the normal user account
  ```
  cp centos7-gnu-cuda9.img /home/jcchen/
  cd /home/jcchen/ && chown jcchen:jcchen centos7-gnu-cuda9.img
  ## Go back to normal user mode
  singularity shell centos7-gnu-cuda9.img
  singularity exec --nv centos7-gnu-cuda9.img which nvidia-smi
  ```
- Example script to run HPCG by using container (24 CPU cores with 2 GPUs per node)
  ```
  export SINGULARITYENV_LD_LIBRARY_PATH=/home/jcchen/openmpi-1.10.7-cuda9-gnu4/lib:$SINGULARITYENV_LD_LIBRARY_PATH
  export SINGULARITYENV_OMP_NUM_THREADS=12
  mpirun -np 2 singularity exec --nv centos7-gnu-cuda9.img /home/jcchen/software/xhpcg-3.1.cuda9.x
  ```
 
 TIPS:
 - Does Singularity support containers that require GPUs? Yes, try this magic but experimental --nv option. The option "--nv" allows you to leverage host GPUs without installing system level drivers into your container.
 - If you intend to run the exising binary outside the container, you DO NOT NEED to install MPI. Howerver, if you would like to compile and install MPI programs within the container, you need to install it. You will need to make sure the correct drivers and libraries between host and container image. [See FAQ](http://singularity.lbl.gov/faq#why-do-we-call-mpirun-from-outside-the-container-rather-than-inside)
 
 References
 - http://singularity.lbl.gov/faq
 - http://singularity.lbl.gov/docs-exec#a-gpu-example 
 - http://singularity.lbl.gov/docs-hpc 
 - http://singularity.lbl.gov/tutorial-gpu-drivers-open-mpi-mtls 

## Create Ubuntu image on CentOS7 platform (in the superuser mode)
- Grab the image of Ubuntu 16.04 LTS and save it as the directory type
```
singularity build --sandbox ubuntu16-pandas/ docker://ubuntu:xenial
```
- Enter the shell of the container image (Ubuntu 16.04)
```
singularity shell --writable ubuntu16-pandas/
```
- Update the OS and Install the packages in the container's shell
```
apt-get update
apt-get install python python-pip 
export LC_ALL=C
pip install --upgrade pip
pip install pandas numpy scipy
```
- Pack the working directory to a single compressed image
```
singularity build ubuntu1604-pandas.img ubuntu16-pandas/
```
- Copy the file to the user's directory
```
mv ubuntu1604-pandas.img /home/jcchen/
chown jcchen:jcchen ubuntu1604-pandas.img
```
6. Testing (in normal user mode)
```
singularity shell ubuntu1604-pandas.img
```
7. Example of the bash sricpt to run python code by using the container image
```
#!/bin/bash
if [ -n $1 ]
then
        singularity exec ubuntu1604-pandas.img python $1
else
        echo "Usage:$ ./run.sh RUN-my.py"
fi
```

