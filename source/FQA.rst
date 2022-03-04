Frequently Asked Questions
************************************

**Restart an interrupted computing task？**
===============================================

The BDF supports breakpoints for a selection of common tasks, including:
  
1. SCF single point energy：Use the ``guess`` keyword to read the molecular orbital of the last SCF iteration of the interrupted task as the initial guess. Specifically, just specify the first guess as ``readmo`` in the $scf module, and rerun the input file.

  .. code-block:: bdf

    $scf
    ...
    guess
     readmo
    $end


2. TDDFT single-point energy：When the TDDFT task is interrupted and idiag is not equal to 2, the TDDFT excitation vector of the last TDDFT iteration (Davidson iteration when idiag=1, iVI iteration when idiag=3) of the task can be read as a first guess. In this case, when idiag=3, only the computation with C(1) symmetry allows for breakpoint sequencing.

   The way to continue the calculation of the TDDFT task breakpoint is as follows:：The TDDFT task is broken by reading the converged SCF wave function of the interrupted task with the ``guess`` keyword in the $scf module and specifying the TDDFT excitation vector of the interrupted task with ``iguess`` in the $tddft module. Assume that the input file is
   
   .. code-block:: bdf
   
     $scf
     ...
     $end
     
     $tddft
     ...
     iguess
      21
     $end

   where the tens digit 2 of iguess=21 indicates the selection of the tight-binding initial guess (but this is not necessary for this example, i.e. the following discussion also applies to iguess=1 or iguess=11), and the single digit 1 indicates that the TDDFT excitation vector for each step is written to a file (when idiag=1, the excitation vector is saved (when idiag=1, the excitation vector is stored in a file with the suffix .dvdsonvec*; when idiag=3, the excitation vector is stored in a file with the suffix .tdx). If the task is interrupted, the breakpoint will be continued by changing the input file to:

   .. code-block:: bdf
   
     $scf
     ...
     guess
      readmo
     $end
     
     $tddft
     ...
     iguess
      11 # or 10, if the user is sure that the job will not be interrupted again
     $end

   where ``guess readmo`` is to read the tracks of the previous SCF iteration, thus avoiding wasting time to re-run the SCF iteration, and iguess=11 with a decimal 1 means to read the TDDFT excitation vector from within the .dvdsonvec* or .tdx file as a first guess. See  :doc:`tddft`  subsection for details on the meaning of the various values of iguess.
   
   Note that: (1) due to the characteristics of the Davidson and iVI methods, the number of TDDFT iterations saved by the above methods is less than the number of iterations saved by SCF breakpoint sequencing under the same conditions, so unless the TDDFT iteration of the previously interrupted task is close to convergence, breakpoint sequencing may not save as many iterations as computing from scratch; (2) TDDFT does not by default The TDDFT excitation vector in the iteration is saved to the hard disk; you must specify iguess of 1, 11 or 21 to save the TDDFT excitation vector in the iteration to the hard disk. If the previously interrupted computation does not contain this keyword, it is not possible to perform a breakpoint continuation. The main reason why the program does not save the excitation vector for each step by default is that the resulting hard disk read/write time may not be negligible, so the user needs to weigh the need to specify saving the excitation vector when performing the calculation.

3. Structure Optimization: just add the ``restart`` keyword to the $compass module, see :doc:`compass` subsection.
4. Numerical Frequency Calculation：add ``restarthess`` keyword in the $bdfopt module, see the :doc:`bdfopt` subsection.

**How is BDF referenced？**
=================================

The first step in using a BDF is to cite the original text of the BDF program :cite:`doi:10.1007/s002140050207,doi:10.1063/1.5143173,doi:10.1142/S0219633603000471,doi:10.1142/9789812794901_0009` 。In addition to this, the different functions of a BDF should also be used with references to the corresponding method's article, see the section on  :doc:`Cite` instructions.

**False excitation energy/complex excitation energy problem for TDDFT calculations**
======================================================================================

If the ground state wave function is unstable or if the SCF converges to a state that is not the true ground state, the TDDFT calculation will suggest the appearance of false excitation energy and, rarely, even complex excitation energy. The false and complex excitation energies have no physical significance. When using the Davidson method, the program gives a warning **Warning: Imaginary Excitation Energy!** and gives the modes of all imaginary/complex excitation energies after convergence of the iterations; when using the iVI method, the program gives a warning **Error in ETDVSI: ABBA mat is not positive! Suggest to use nH-iVI.**, and the subsequent calculation does not try to continue solving for the imaginary/complex excitation energy, but only for the real excitation energy (so when using the iVI method, one cannot conclude that the system does not have excited states with imaginary/complex excitation energy just because the final converged excitation energy is all real). In this case, the ground state wave function should be re-optimized to find a stable solution, or the TDA should be used to calculate the excitation energy.

**Available Memory and Computational Efficiency of J and K Operators for TDDFT**
==================================================================================

The keyword **MEMJKOP** of TDDFT module can be used to set the maximum available memory for TDDFT to compute J and K operators if the number of roots required to be solved by TDDFT is large and the default memory of the program is not enough, which makes TDDFT computation less efficient. For example, if **4** roots are required to be computed, TDDFT gives the following output.

.. code-block:: bdf

     Maximum memory to calculate JK operator:        1024.000 M
     Allow to calculate    2 roots at one pass for RPA ...
     Allow to calculate    4 roots at one pass for TDA ...

The maximum available memory for calculating JK operators is **1024M**, where the unit is megabytes (MB). In case of RPA (i.e. TDDFT) calculations, 2 roots are allowed per integration calculation, and 4 roots are allowed for TDA calculations. If the user requires TDA calculation, one integration calculation will get JK operators of all roots, and RPA calculation needs to calculate the integration twice, which reduces the calculation efficiency. You can set ``MEMJKOP`` to 2048MB to increase the memory so that only one integration is computed for each iteration step. Note that the actual physical memory used is about **2048MB*OMP_NUM_THREADS** , i.e. it needs to be multiplied by the number of OpenMP threads.

**Calculating segmentation fault with available stack area memory**
=====================================================================

If a **segmentation fault** occurs in a BDF calculation, most of the time it is caused by the user not having enough memory available in the stack area, and under Linux, the available stack area memory size can be set with the  **ulimit** command.

First enter the command：

.. code-block:: bdf 

  $ulimit -a

The output prompt is as follows：

.. code-block:: bdf

    core file size          (blocks, -c) 0
    data seg size           (kbytes, -d) unlimited
    scheduling priority             (-e) 0
    file size               (blocks, -f) unlimited
    pending signals                 (-i) 256378
    max locked memory       (kbytes, -l) 64
    max memory size         (kbytes, -m) unlimited
    open files                      (-n) 4096
    pipe size            (512 bytes, -p) 8
    POSIX message queues     (bytes, -q) 819200
    real-time priority              (-r) 0
    stack size              (kbytes, -s) 4096 
    cpu time               (seconds, -t) unlimited
    max user processes              (-u) 256378
    virtual memory          (kbytes, -v) unlimited
    file locks                      (-x) unlimited

Here  ``stack size              (kbytes, -s) 4096`` means that the user has 4096KB of memory available in the stack area, which is only 4 megabytes, and can be accessed with the command

.. code-block:: bdf

    ulimit -s unlimited

Set an unlimited amount of stack area memory available to the user. Many Linux distributions have limits on the stack area. Strictly speaking, 
there is **hard limit** and **soft limit** on the size of the stack area memory limit, and normal users only have permission to set the stack area memory 
smaller than the **hard limit** . If ``ulimit -s unlimited`` prompts an error

.. code-block:: bdf

    $ulimit -S
    -bash: ulimit: stack size: cannot modify limit: Operation not permitted

You need to change the **hard limit** of memory in the available stack area with your root account or contact your system administrator to resolve the issue.

**OpenMP Parallel Computing**
=================================================================

BDF supports OpenMP parallel computing and requires the number of available OpenMP threads to be set in the run script as follows：

.. code-block:: bdf

    export OMP_NUM_THREADS=8

Here is set up to have a maximum of 8 OpenMP threads available for parallel computing.

**OpenMP's stack area memory size**
=================================================================

Intel compiler available stack area memory, especially when using OpenMP parallel computing, the intel compiler puts dynamic memory from the parallel area into the stack area to obtain higher computational efficiency. Therefore, the user needs to set the size of the available stack area memory for OpenMP in the BDF run script as follows：

.. code-block:: bdf

    export OMP_STACKSIZE=2048M

Here the available stack area memory size of OpenMP is set to 2048MB. Note: If you use OpenMP for multi-thread parallelism, the total heap memory used by the system is **OMP_STACKSIZE*OMP_NUM_THREADS** 。

.. important::
  The environment variable OMP_STACKSIZE is a general environment variable and has an overlay relationship with the special environment variables of other OpenMP runtime libraries：

  KMP_STACKSIZE（Intel OpenMP） > GOMP_STACKSiZE（GNU OpenMP） > OMP_STACKSIZE

  Therefore, if a higher priority environment variable is set in the script, the value of OMP_STACKSIZE will be overwritten.

**Intel 2018 Edition Fortran Compiler**
=================================================================

The Intel 2018 version of the Fortran compiler is buggy and should be avoided for compiling BDFs.


**SCF non-convergence**
=================================================================

See the section on :ref:`dealing with non-convergence in self-consistent field calculations<SCFConvProblems>` in the :doc:`SCFTech` chapter

**SCF energy is far below the expected value (more than 1 Hartree below the expected value) or SCF energy is displayed as a string of asterisks**
====================================================================================================================================================

This is usually caused by the basis group linear correlation problem. See the discussion of basis group linear correlation problems in the section on dealing with nonconvergence 
in self-consistent field calculations in the :doc:`SCFTech` chapter. Note that while this section focuses on :ref:`solutions to problems where the SCF does not converge<SCFConvProblems>` due to basis group linear 
correlation problems, these methods are also applicable in cases where the basis group linear correlation problem only causes errors in the SCF energy and does not lead to non-convergence.

**How to use custom base groups**
=================================================================

See the section :ref:`Custom Basis File<SelfdefinedBasis>` in :doc:`Gaussian-Basis-Set` .

