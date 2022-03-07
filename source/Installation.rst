Installation and Operation
************************************

.. attention::

   Ordinary users do not need to read the installation and compilation related content, and can directly skip to :ref:`BDF program running<run-bdfpro>` and :ref:`BDF graphical interface<run-bdfgui>` 。


Installation instructions
================================================

Hardware environment
-------------------------------------------------
In principle, BDF can be compiled and installed on any Unix and Unix-like operating system. We have tested compilation on some common hardware and software environments. For other hardware platforms, users may encounter some problems due to limitations and defects of the operating system and compiler version. In most cases, users can eventually compile and install BDF software successfully by setting the correct compiler flags and system software paths according to their hardware environment.

To compile the BDF software, one needs at least 2 GB of free disk space, depending on the installation method (e.g., CMake keeps the target file), and the actual size after compilation is about 1.3 GB. To run the test cases of the BDF, one should provide at least 1 GB of disk space for caching the intermediate data of the calculation, depending on the size of the calculation system and the integration method used. The amount of cache space required depends on the size of the computational system and the integration method used (the disk space required for the direct integration algorithm is much smaller than for the traditional mode of storing two-electron integrals). In general, for calculations using the direct integration algorithm, more than 4 GB of disk space should be provided for data caching.

Software environment configuration
------------------------------------------------------------------------

The minimum requirements for compilers and mathematical libraries for direct compilation and installation from the source code of a BDF are：

 * CMake version 3.15 and above (compile with cmake)
 * C++ compiler (supports C++03 and above syntax)
 * C compiler
 * BLAS/LAPACK math library, the interface needs to be a 64-bit integer
 * CMake version 3.15 and above (compile with cmake)
 * Python 2.7 and above. Python 2 is not compatible with Python 3, and Python 3 is not yet fully adapted
 
Usually use GCC 4.6 and above to compile normally.

Optional:
  * C/C++, Fortran compiler for Intel Parallel Studio XE Cluster
  * Optimized BLAS/LAPACK library (such as Intel's MKL, AMD's ACML, OpenBLAS, etc.)
  * To compile the parallel version of BDF, Openmpi 1.4.1 or above is required
  * Compiling BDF for GPU requires OpenCL 1.5 or above, and AMD's Rocm or Nvidia's Cuda

Cmake compile BDF
==========================================================================

1. Intel FORTRAN compiler and GNU GCC / G + + compiler are mixed, linked to MKL math library, and OpenMP parallel support
-------------------------------------------------------------------------------------------------------------------------------

.. code-block:: shell

    # Set up the compiler
    $ export FC=ifort
    $ export CC=gcc
    $ export CXX=g++
    # cmake is automatically executed by the setup command
    $ ./setup --fc=${FC} --cc=${CC} --cxx=${CXX} --bdfpro --omp --int64 --mkl sequential $1
    # Build the BDF in the build directory
    $ cd build
    # Use the make command to compile BDF, specifying 4 CPUs in parallel with the -j4 parameter
    $ make -j4
    # Install BDF
    $ make install
    # After copying BDF PKG Pro under build to any path, write the correct path in bdfrc, such as:
    $ BDFHOME=/home/user/bdf-pkg-pro
    # Run command
    $ $BDFHOME/sbin/bdfdrv.py -r **.inp

2. GNU compiler gfortran/gcc/g++, links to MKL mathematical libraries, OpenMP parallel support
--------------------------------------------------------------------------------------------------------

.. code-block:: shell

    # Set up the compiler
    $ export FC=gfortran
    $ export CC=gcc
    $ export CXX=g++
    # cmake is automatically executed by the setup command
    $ ./setup --fc=${FC} --cc=${CC} --cxx=${CXX} --bdfpro --omp --int64 --mkl sequential $1
    # Build the BDF in the build directory
    $ cd build
    # Use the command make to compile BDF, specifying 4 CPUs in parallel with the -j4 parameter 
    $ make -j4
    # Install BDF
    $ make install
    # After copying BDF PKG Pro under build to any path, write the correct path in bdfrc, such as:
    $ BDFHOME=/home/user/bdf-pkg-pro
    # Run command
    $ $BDFHOME/sbin/bdfdrv.py -r **.inp

3. Intel compiler ifort/icc/icpc, linking MKL mathematical libraries, OpenMP parallel support
-------------------------------------------------------------------

.. code-block:: shell

    # Set up the complier
    $ export FC=ifort
    $ export CC=icc
    $ export CXX=icpc
    # cmake is automatically executed by the setup command
    $ ./setup --fc=${FC} --cc=${CC} --cxx=${CXX} --bdfpro --omp --int64 --mkl sequential $1
    # Build the BDF in the build directory
    $ cd build
    # Use the command make to compile BDF, specifying 4 CPUs in parallel with the -j4 parameter 
    $ make -j4
    # Install BDF
    $ make install
    # After copying BDF PKG Pro under build to any path, write the correct path in bdfrc, such as:
    $ BDFHOME=/home/user/bdf-pkg-pro
    # Run command
    $ $BDFHOME/sbin/bdfdrv.py -r **.inp

.. Warning::
   1. Gcc compiler version 9.0 and above, mixed with Intel FORTRAN compiler, link program error, because the OpenMP version of Intel FORTRAN compiler lags behind GNU compiler. Therefore, GNU 9.0 and above compilers currently do not support mixed compilation of GNU and Intel compilers.
   2. Intel FORTRAN version 2018 compiler has many bugs and should be avoided.

4  Compile bdfpro and require to generate HZW license file
-------------------------------------------------------------------

The main steps are the same as those in the previous three cases. When running the setup command, you need to add a parameter ``--hzwlic``, such as: 

.. code-block:: bdf

    #Cmake is automatically executed by the setup command
    $./setup --fc=${FC} --cc=${CC} --cxx=${CXX} --bdfpro --hzwlic --omp --int64 --mkl sequential $1

After running the  ``make install`` command, the following output will be given: 

.. code-block:: bdf

    Please run command '/home/bsuo/bdf-pkg-pro/bdf-pkg-pro/bin/hzwlic.x /home/bsuo/bdf-pkg-pro/build/bdf-pkg-pro' to generate HZW license!

Here, ``/home/bsuo/bdf-pkg-pro`` is the bdfpro source file directory, ``/home/bsuo/bdf-pkg-pro/build/bdf-pkg-pro`` is the binary installation directory of bdfpro. Run command:

.. code-block:: bdf

    /home/bsuo/bdf-pkg-pro/bdf-pkg-pro/bin/hzwlic.x /home/bsuo/bdf-pkg-pro/build/bdf-pkg-pro

After that, **LicenseNumber.txt** is generated in the ``/home/bsuo/bdf-pkg-pro/build/bdf-pkg-pro/license`` directory.


.. _run-bdfpro:

Program operation
==========================================================================

BDF needs to run under a Linux terminal. To run BDF, one needs to prepare an input file, the exact format of which is described in later sections of the manual. The BDF installation directory under tests/input contains some BDF input examples. Here we will use the test cases that come with BDF as an example and briefly explain how to run BDF first.

Running BDF will use a number of environment variables：

.. table::

   =======================  ===================================================================================== ========================== 
    Environment variable     Instructions                                                                           Must be set or not
    BDFHOME                  Specify the installation directory of BDF                                              Yes     
    BDF_WORKDIR              The working directory of BDF, that is, the execution directory of the current task     No, set automatically                      
    BDF_TMPDIR               Specifies the cache file storage directory for BDF                                     Yes          
    BDFTASK                  BDF calculation task name, if entered as h2o.inp, task name is h2o                     No, set automatically         
   =======================  ===================================================================================== ========================== 

Run BDF standalone and execute the job with a shell script
------------------------------------------------------------
Assuming that the user directory is /home/user, BDF is installed in /home/user/bdf-pkg-pro. After prepare the input file ``ch2-hf.inp`` ,you need to prepare a shell script and enter the following

.. code-block:: shell

    #!/bin/bash

    export BDFHOME=/home/user/bdf-pkg-pro
    export BDF_WORKDIR=./
    export BDF_TMPDIR=/tmp/$RANDOM

    ulimit -s unlimited
    ulimit -t unlimited

    export OMP_NUM_THREADS=4
    export OMP_STACKSIZE=512M 

    $BDFHOME/sbin/bdfdrv.py -r $1

Name the script run.sh, use "chmod +x run.sh" to give permission to execute the script, and then execute it as follows.

.. code-block:: shell

    # Create a new folder named test in /home/user
    $ mkdir test
    $ cd test
    # Copy /home/user/bdf-pkg-pro/tests/easyinput/ch2-hf.inp to test folder
    $ cp /home/user/bdf-pkg-pro/tests/easyinput/ch2-hf.inp
    # Run the submit command in the test directory
    $ ./run.sh ch2-hf.inp &> ch2-hf.out&

.. hint::
    When BDF prints the output to standard output, you need to use the redirection command ``>`` to direct to the file ch2-hf.out.

Submitting BDF jobs using the PBS job management system
----------------------------------------------------------

An example script for a PBS submission BDF job is as follows：

.. code-block:: shell

    #!/bin/bash
    #PBS -N jobname
    #PBS -l nodes=1:ppn=4
    #PBS -l walltime=1200:00:00
    #PBS -q batch
    #PBS -S /bin/bash
    
    #### Set the environment variables #######
    #module load tools/openmpi-3.0.1-intel-socket

    #module load compiler/intel-compiler-2020
    
    #### Set the PATH to find your applications #####
    export BDFHOME=/home/bbs/bdf-pkg-pro
    
    # Specify the temporary file storage directory where BDF runs
    export BDF_TMPDIR=/tmp/$RANDOM
    
    # Specify the Stack memory size for OpenMP
    export OMP_STACKSIZE=2G
    
    # Specify the number of available OpenMP threads, which should be equal to the number defined by ppn
    export OMP_NUM_THREADS=4
    
    #### Do not modify this section ! #####
    cd $PBS_O_WORKDIR
    
    $BDFHOME/sbin/bdfdrv.py -r jobname.inp


Submit BDF jobs using Slurm job management system
----------------------------------------------------

An example script for slurm to submit a BDF job is as follows：

.. code-block:: shell

    #!/bin/bash
    #SBATCH --partition=v6_384
    #SBATCH -J bdf.slurm
    #SBATCH -N 1
    #SBATCH --ntasks-per-node=48

    
    #### Set the environment variables #######
    #module load tools/openmpi-3.0.1-intel-socket
    #module load compiler/intel-compiler-2020
    
    #### Set the PATH to find your applications #####
    export BDFHOME=/home/bbs/bdf-pkg-pro
    
    # Specify the temporary file storage directory where BDF runs
    export BDF_WORKDIR=./
    export BDF_TMPDIR=/tmp/$RANDOM
    
    # Specify the Stack memory size for OpenMP
    export OMP_STACKSIZE=2G
    
    # Specify the number of available OpenMP threads, which should be equal to the number defined by ppn
    export OMP_NUM_THREADS=4
    
    #### Do not modify this section ! #####
    $BDFHOME/sbin/bdfdrv.py -r jobname.inp



.. important::
    1. The problem with stacksize.The Intel Fortran compiler requires a large amount of stack memory for programs to run, and the default stacksize is usually too small and needs to be specified by ``ulimit -s unlimited`` .
    2. OpenMP Number of threads in parallel. OMP_NUM_THREADS is used to set the number of threads in parallel for OpenMP. BDF relies on OpenMP parallelism to improve computational efficiency. If you are using the Bash Shell, you can use the command ``export OMP_NUM_THREADS=N`` to specify the number of OpenMP parallel threads to use. to use N OpenMP threads to accelerate the computation.
    3. OpenMP available heap memory, users can use ``export OMP_STACKSIZE=1024M`` to specify the size of the heap memory available to each thread of OpenMP, and the total heap memory size is ``OMP_STACKSIZE*OMP_NUM_THREADS`` .



QM/MM Computing Environment Configuration
-------------------------------------------------
.. _qmmmsetup:

We recommend using Anaconda to manage and configure the QM/MM computing environment ( `see <https://www.anaconda.com>`_ ).

*  Configure the runtime environment in anaconda

.. code-block:: shell

  conda create –name yourEnvname python=2.7
  conda activate yourEnvname
  #Configure Cython and PyYAML
  conda install pyyaml #or pip install pyyaml
  conda install cython 

*  Installation and configuration of pDynamo-2

BDF pDynamo-2 has been built into the sbin directory of the installation directory, so run the following commands in the sbin directory to install and configure in the sbin directory to install and configure：

.. code-block:: shell

  cd pDynamo_2.0.0
  cd installation
  python ./install.py

After the installation script is run, two environment configuration files, environment_bash.com，environment_cshell.com are generated. Users can load this environment file in their ``.bashrc`` via source to set up the runtime environment. 

.. note::

  The compilation process automatically selects the C compiler, for MAC systems it is recommended to install the GCC compiler using ``homebrew`` and add CC=gcc-8. Other versions of the gcc compiler correspond to gcc-6 or gcc-7, etc. Other versions of the gcc compiler correspond to gcc-6 or gcc-7, respectively. The version above gcc-8 is not tested at the moment.
  
When pDynamo-2 is run, the ``qmmmrun.sh`` file in the sbin directory is called by default to perform QM calculations. When configuring the environment, you need to make sure that the sbin directory is in the system PATH. You can add it with the following command.

.. code-block:: shell

  export PATH=/BDFPATH/sbin:$PATH

*  The final step is to specify the BDF program temporary file storage folder, either by running the following command, or by setting the variable in the environment variable in the environment variable.

.. code-block:: shell
  
  export PDYNAMO_BDFTMP=YourBDF_tmpPATH

To check if pDynamo is installed correctly, you can run the examples that come with the software, which are located in the  **pDynamo_2.0.0/book/examples** directory, by running the following command

.. code-block:: shell

  python RunExamples.py
