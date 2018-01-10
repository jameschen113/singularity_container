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
  '''
  export SINGULARITYENV_LD_LIBRARY_PATH=/home/jcchen/openmpi-1.10.7-cuda9-gnu4/lib:$SINGULARITYENV_LD_LIBRARY_PATH
  export SINGULARITYENV_OMP_NUM_THREADS=12
  mpirun -np 2 singularity exec --nv centos7-gnu-cuda9.img /home/jcchen/software/xhpcg-3.1.cuda9.x
  '''
  
 References
 - http://singularity.lbl.gov/faq
 - http://singularity.lbl.gov/docs-exec#a-gpu-example 
 - http://singularity.lbl.gov/docs-hpc 
 - http://singularity.lbl.gov/tutorial-gpu-drivers-open-mpi-mtls 
