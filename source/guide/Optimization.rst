.. _GeomOptimization:

Structural Optimization and Frequency Calculation
===================================================

The purpose of structural optimization is to find the minimum point of the potential energy surface of the system. The minimum point that can be found depends
on the initial structure provided in the input file. The closer to the minima, the easier it is to converge to the minima.

Structural optimization is mathematically equivalent to the problem of finding the extrema of a multivariate function.：

.. math::
    F_{i} = -\frac{\partial E(R_1,R_2,\dots,R_N)}{\partial R_i} = 0, i=1,2,\dots,N

Commonly used algorithms for structural optimization are as follows：

#. Steepest descent：The steepest descent method is a line search along the direction of the negative gradient, which is very efficient for structures far from the minima, but slow to converge near the minima and easy to oscillate.
#. Conjugate gradient：Conjugate gradient method is a modification of the most rapid descent method, the optimization direction of each step and the previous step of the combination of optimization direction, can somewhat alleviate the problem of oscillation
#. Newton method：The idea of Newton method is to expand the function with respect to the current position in the Taylor series. Newton method converges quickly, for the quadratic function can go to the minimum in one step. However, the Newton method requires solving the Hessian matrix, which is very expensive to compute, and the quasi-Newton method is generally used in geometric optimization.
#. Quasi-Newton method：The quasi-Newton method constructs the Hessian matrix by approximation, and the Hessian matrix of the current step is obtained based on the forces of the current step and the Hessian matrix of the previous step. There are various approaches, the most commonly used is the BFGS method, in addition to DFP, MS, PSB, etc. Since the Hessian of the quasi-Newton method is constructed approximately, the accuracy of each optimization step is lower than that of the Newton method, and the number of steps required to reach convergence is more than that of the Newton method. However, the total optimization time is still significantly reduced because each step takes much less time.

The structural optimization of the BDF is implemented by the BDFOPT module, which supports the Newtonian and quasi-Newtonian based methods for minima and transition
The BDFOPT module supports Newtonian and quasi-Newtonian methods for the optimization of minima and transition structures, and supports restricted
structure optimization. The following is an example of the input file format of the BDFOPT module, and an explanation of the output file。

Base state optimization: Optimization of monochloromethane（ :math:`\ce{CH3Cl}` ）at the B3LYP/def2-SV(P) level
---------------------------------------------------------------------------------------------------------------------

.. code-block:: bdf

    $compass
    title
       CH3Cl geomopt
    basis
       def2-SV(P)
    geometry
     C                  2.67184328    0.03549756   -3.40353093
     H                  2.05038141   -0.21545378   -2.56943947
     H                  2.80438882    1.09651909   -3.44309081
     H                  3.62454948   -0.43911916   -3.29403269
     Cl                 1.90897396   -0.51627638   -4.89053325
    end geometry
    $end

    $bdfopt
    solver
     1
    $end

    $xuanyuan
    $end

    $scf
    rks
    dft
     b3lyp
    $end

    $resp
    geom
    $end

The RESP module is responsible for calculating the DFT gradient. Unlike most other tasks, in the structure optimization task, the program does not sequentially and
single, linear invocation of the modules, but rather, the modules are invoked several times iteratively. The specific sequence of calls is as follows：

 1. run COMPASS, which reads the molecular structure and other information； 
 2. run BDFOPT to initialize the intermediate quantities needed for structure optimization；
 3. BDFOPT starts a separate BDF process to calculate the energy and gradient under the current structure, which only executes COMPASS, XUANYUAN, SCF, and RESP modules and skips BDFOPT. i.e., most of the time the user will find two BDF processes running independently of each other, one of which is the process to which BDFOPT belongs and is waiting. is in the waiting state, while the other process is performing energy and gradient calculations. To avoid cluttering the output file, the output of the latter process is automatically redirected to a file with the ``.out.tmp`` suffix, thus separating it from the output of the BDFOPT module (which is typically redirected by the user to an ``.out`` file)
 4. when the latter process is finished, BDFOPT aggregates the energy and gradient information of the current structure and adjusts the molecular structure accordingly with a view to reducing the energy of the system；
 5. BDFOPT determines whether the structure converges according to the gradient of the current structure and the size of the current geometry step, if it converges, or if the structure optimization reaches the maximum number of iterations, the program ends; if it does not converge, the program jumps to step 3。

Therefore, the ``.out`` file contains only the output of COMPASS and BDFOPT modules, which can be used to monitor the process of structural optimization, but does not
contain information on SCF iterations, Buju analysis, etc., which needs to be viewed in the ``.out.tmp`` file

Taking the structure optimization task of :math:`\ce{CH3Cl}` above as an example, the output of the BDFOPT module in the ``.out`` file can be seen as follows:

.. code-block:: 

       Geometry Optimization step :    1

      Single Point SCF for geometry optimization, also get force.


     ### [bdf_single_point] ### nstate= 1
     Allow rotation to standard orientation.

     BDFOPT run - details of gradient calculations will be written
     into .out.tmp file.

    ...

    ### JOB TYPE = SCF ###
    E_tot= -499.84154693
    Converge= YES

    ### JOB TYPE = RESP_GSGRAD ###
    Energy= -499.841546925072
         1        0.0016714972        0.0041574983       -0.0000013445
         2       -0.0002556962       -0.0006880567        0.0000402277
         3       -0.0002218807       -0.0006861734       -0.0000225761
         4       -0.0003229876       -0.0006350885       -0.0000059774
         5       -0.0008670369       -0.0021403962       -0.0000084046

It can be seen that BDFOPT calls the BDF program itself to calculate the SCF energy and gradient of the molecule under the initial guess structure. the detailed
output of the SCF and gradient calculations is in the ``.out.tmp`` file, while the ``.out`` file only extracts the energy and gradient values, as well as information on
whether the SCF converges or not. The unit of energy is Hartree and the unit of gradient is Hartree/Bohr.


Since BDFOPT is a structure optimized in redundant internal coordinates（ ``solver`` = 1），in order to generate the molecular structure for the next step, the redundant
internal coordinates of the molecule must be generated first. Therefore, in the first step of the structure optimization, the output file also gives the definition
of the individual redundant internal coordinates (i.e. the atomic numbers involved in the formation of the corresponding bonds, bond angles and dihedral angles), as well as their values (bond lengths in angstroms, bond angles in degrees).

.. code-block:: 

    |******************************************************************************|
           Redundant internal coordinates on Angstrom/Degree

      Name         Definition         Value     Constraint
      R1          1   2               1.0700    No
      R2          1   3               1.0700    No
      R3          1   4               1.0700    No
      R4          1   5               1.7600    No
      A1          2   1   3           109.47    No
      A2          2   1   4           109.47    No
      A3          2   1   5           109.47    No
      A4          3   1   4           109.47    No
      A5          3   1   5           109.47    No
      A6          4   1   5           109.47    No
      D1          4   1   3   2      -120.00    No
      D2          5   1   3   2       120.00    No
      D3          2   1   4   3      -120.00    No
      D4          3   1   4   2       120.00    No
      D5          5   1   4   2      -120.00    No
      D6          5   1   4   3       120.00    No
      D7          2   1   5   3       120.00    No
      D8          2   1   5   4      -120.00    No
      D9          3   1   5   2      -120.00    No
      D10         3   1   5   4       120.00    No
      D11         4   1   5   2       120.00    No
      D12         4   1   5   3      -120.00    No

    |******************************************************************************|

After the molecular structure has been updated, the program calculates the gradient as well as the size of the geometric step and determines whether the structural optimization converges：

.. code-block:: 

                           Force-RMS    Force-Max     Step-RMS     Step-Max
        Conv. tolerance :  0.2000E-03   0.3000E-03   0.8000E-03   0.1200E-02
        Current values  :  0.8833E-02   0.2235E-01   0.2445E-01   0.5934E-01
        Geom. converge  :     No           No           No           No

The program considers the convergence of the structural optimization only when the current values of Force-RMS, Force-Max, Step-RMS, and Step-Max are less than the corresponding convergence limits (i.e.,  ``Geom. converge`` column is Yes).
For this example, the structural optimization converges at step 5, when the output message not only contains the values of the convergence criteria, but also
explicitly informs the user that the geometry optimization has converged, and prints the converged molecular structure in Cartesian and internal coordinates, respectively.

.. code-block:: 

        Good Job, Geometry Optimization converged in     5 iterations!

       Molecular Cartesian Coordinates (X,Y,Z) in Angstrom :
          C          -0.93557703       0.15971089       0.58828595
          H          -1.71170348      -0.52644336       0.21665897
          H          -1.26240747       1.20299703       0.46170050
          H          -0.72835075      -0.04452039       1.64971607
          Cl          0.56770184      -0.09691413      -0.35697029

                           Force-RMS    Force-Max     Step-RMS     Step-Max
        Conv. tolerance :  0.2000E-03   0.3000E-03   0.8000E-03   0.1200E-02
        Current values  :  0.1736E-05   0.4355E-05   0.3555E-04   0.6607E-04
        Geom. converge  :     Yes          Yes          Yes          Yes


      Print Redundant internal coordinates of the converged geometry

    |******************************************************************************|
           Redundant internal coordinates on Angstrom/Degree

      Name         Definition         Value     Constraint
      R1          1   2               1.1006    No
      R2          1   3               1.1006    No
      R3          1   4               1.1006    No
      R4          1   5               1.7942    No
      A1          2   1   3           110.04    No
      A2          2   1   4           110.04    No
      A3          2   1   5           108.89    No
      A4          3   1   4           110.04    No
      A5          3   1   5           108.89    No
      A6          4   1   5           108.89    No
      D1          4   1   3   2      -121.43    No
      D2          5   1   3   2       119.28    No
      D3          2   1   4   3      -121.43    No
      D4          3   1   4   2       121.43    No
      D5          5   1   4   2      -119.28    No
      D6          5   1   4   3       119.29    No
      D7          2   1   5   3       120.00    No
      D8          2   1   5   4      -120.00    No
      D9          3   1   5   2      -120.00    No
      D10         3   1   5   4       120.00    No
      D11         4   1   5   2       120.00    No
      D12         4   1   5   3      -120.00    No

    |******************************************************************************|

Note that the convergence limits for the root-mean-square force and the rootmean-square step can be set here using the ``tolgrad`` and ``tolstep`` keywords,
respectively, and the program automatically adjusts the convergence limits for the maximum force and the maximum step according to the set values; when using
the DL-FIND library (see later), the energy convergence limit can also be specified by ``tolene``. However, it is generally not recommended to adjust the convergence limits by the user.


At the same time, the program generates files with the suffix ``.optgeom`` , which contain the Cartesian coordinates of the converged molecular structure, but in Bohr units instead of Angstrom:

.. code-block:: 

    GEOM
            C             -0.7303234729        -2.0107211546        -0.0000057534
            H             -0.5801408002        -2.7816264533         1.9257943885
            H              0.4173171420        -3.1440530286        -1.3130342173
            H             -2.7178161476        -2.0052051760        -0.6126883555
            Cl             0.4272106261         1.1761889168        -0.0000021938

The ``.optgeom`` file can be converted to xyz format using the tool ``optgeom2xyz.py`` under ``$BDFHOME/sbin/`` , so that the optimized molecular structure can be viewed in
any visualization software that supports xyz format. For example, if the file to be converted is named filename.optgeom, execute the following command line: (note
that you must first set the environment variable $BDFHOME, or manually replace $BDFHOME in the following command with the path to the BDF folder)

.. code-block:: shell

    $BDFHOME/sbin/optgeom2xyz.py filename

You can get filename.xyz in the current directory.

Frequency calculation: Resonant frequencies and thermochemical quantities of :math:`\ce{CH3Cl}` in the equilibrium structure
-------------------------------------------------------------------------------------------------------------------------------------------------

After convergence of the structure optimization, the frequency analysis can be performed. Prepare the following input file：

.. code-block:: bdf

    $compass
    title
     CH3Cl freq
    basis
     def2-SV(P)
    geometry
     C          -0.93557703       0.15971089       0.58828595
     H          -1.71170348      -0.52644336       0.21665897
     H          -1.26240747       1.20299703       0.46170050
     H          -0.72835075      -0.04452039       1.64971607
     Cl          0.56770184      -0.09691413      -0.35697029
    end geometry
    $end

    $bdfopt
    hess
     only
    $end

    $xuanyuan
    $end

    $scf
    rks
    dft
     b3lyp
    $end

    $resp
    geom
    $end

where the molecular structure is the converged structure obtained from the above structure optimization task. Note that we have added ``hess only`` to the BDFOPT
module, where ``hess`` stands for computed (numerical) Hessian, and the meaning of ``only`` will be described in detail in the subsequent sections. The program perturbs
each atom in the molecule in the positive x-axis, negative x-axis, positive yaxis, negative y-axis, positive z-axis, and negative z-axis directions, and
calculates the gradient under the perturbed structure, e.g.


.. code-block:: 

     Displacing atom    1 (+x)...

     ### [bdf_single_point] ### nstate= 1
     Do not allow rotation to standard orientation.

     BDFOPT run - details of gradient calculations will be written
     into .out.tmp file.

    ...

    ### JOB TYPE = SCF ###
    E_tot= -499.84157717
    Converge= YES

    ### JOB TYPE = RESP_GSGRAD ###
    Energy= -499.841577166026
         1        0.0005433780       -0.0000683370       -0.0000066851
         2       -0.0000516384        0.0000136326       -0.0000206081
         3       -0.0001360377        0.0000872513        0.0000990006
         4       -0.0003058645        0.0000115926       -0.0000775624
         5       -0.0000498284       -0.0000354732        0.0000023346

If the atomic number of the system is N, then a total of 6N gradients have to be calculated. However, in practice the program also calculates the gradients of the
unperturbed structure in order to allow the user to check whether the aforementioned structural optimization has indeed converged, so that the program
actually calculates a total of 6N+1 gradients. Finally, the program obtains the Hessian of the system by the finite difference method.：

.. code-block:: 

    |--------------------------------------------------------------------------------|
              Molecular Hessian - Numerical Hessian (BDFOPT)

                          1              2              3              4              5              6
           1   0.5443095266  -0.0744293569  -0.0000240515  -0.0527420800   0.0127361607  -0.0209022664
           2  -0.0744293569   0.3693301504  -0.0000259750   0.0124150102  -0.0755387479   0.0935518380
           3  -0.0000240515  -0.0000259750   0.5717632089  -0.0213157291   0.0924260912  -0.2929392390
           4  -0.0527420800   0.0124150102  -0.0213157291   0.0479752418  -0.0069459473   0.0239610358
           5   0.0127361607  -0.0755387479   0.0924260912  -0.0069459473   0.0867377886  -0.0978524147
           6  -0.0209022664   0.0935518380  -0.2929392390   0.0239610358  -0.0978524147   0.3068416997
           7  -0.1367366097   0.0869338594   0.0987840786   0.0031968314  -0.0034098009  -0.0016497426
           8   0.0869913627  -0.1185605401  -0.0945336434  -0.0070787068   0.0099076105   0.0045621064
           9   0.0986508197  -0.0953400774  -0.1659434327   0.0163191407  -0.0140134399  -0.0166739137
          10  -0.3054590932   0.0111756577  -0.0774713107   0.0016297078   0.0019657599  -0.0021771884
          11   0.0112823039  -0.0407134661   0.0021058508   0.0106623780   0.0018506067   0.0005120364
          12  -0.0775840113   0.0018141942  -0.0759448618  -0.0275602878   0.0006820252  -0.0059830018
          13  -0.0486857506  -0.0362556088   0.0000641125  -0.0000787206  -0.0045253276   0.0011289985
          14  -0.0360823429  -0.1334063062   0.0000148321  -0.0091074064  -0.0228930763  -0.0010993076
          15   0.0001686252   0.0004961854  -0.0352553706   0.0084860406   0.0189117305   0.0079690194

                          7              8              9             10             11             12
           1  -0.1367366097   0.0869913627   0.0986508197  -0.3054590932   0.0112823039  -0.0775840113
           2   0.0869338594  -0.1185605401  -0.0953400774   0.0111756577  -0.0407134661   0.0018141942
           3   0.0987840786  -0.0945336434  -0.1659434327  -0.0774713107   0.0021058508  -0.0759448618
           4   0.0031968314  -0.0070787068   0.0163191407   0.0016297078   0.0106623780  -0.0275602878
           5  -0.0034098009   0.0099076105  -0.0140134399   0.0019657599   0.0018506067   0.0006820252
           6  -0.0016497426   0.0045621064  -0.0166739137  -0.0021771884   0.0005120364  -0.0059830018
           7   0.1402213115  -0.0861503922  -0.1081442631  -0.0130805143   0.0143574755   0.0192323598
           8  -0.0861503922   0.1322736798   0.1009922720   0.0016534140   0.0024111759   0.0011733340
           9  -0.1081442631   0.1009922720   0.1688786678  -0.0038440081   0.0072277457   0.0091535975
          10  -0.0130805143   0.0016534140  -0.0038440081   0.3186765202  -0.0079165663   0.0838593213
          11   0.0143574755   0.0024111759   0.0072277457  -0.0079165663   0.0509206668  -0.0029665370
          12   0.0192323598   0.0011733340   0.0091535975   0.0838593213  -0.0029665370   0.0707430980
          13   0.0064620333   0.0044161973  -0.0031236007  -0.0026369496  -0.0283860480   0.0017966445
          14  -0.0119743475  -0.0258901434   0.0013817613  -0.0066143965  -0.0145372292  -0.0006143935
          15  -0.0078330845  -0.0126024853   0.0040383425  -0.0008566397  -0.0068931757   0.0018028482

                         13             14             15
           1  -0.0486857506  -0.0360823429   0.0001686252
           2  -0.0362556088  -0.1334063062   0.0004961854
           3   0.0000641125   0.0000148321  -0.0352553706
           4  -0.0000787206  -0.0091074064   0.0084860406
           5  -0.0045253276  -0.0228930763   0.0189117305
           6   0.0011289985  -0.0010993076   0.0079690194
           7   0.0064620333  -0.0119743475  -0.0078330845
           8   0.0044161973  -0.0258901434  -0.0126024853
           9  -0.0031236007   0.0013817613   0.0040383425
          10  -0.0026369496  -0.0066143965  -0.0008566397
          11  -0.0283860480  -0.0145372292  -0.0068931757
          12   0.0017966445  -0.0006143935   0.0018028482
          13   0.0450796910   0.0642866688   0.0000350066
          14   0.0642866688   0.1954779468   0.0000894464
          15   0.0000350066   0.0000894464   0.0213253497

    |--------------------------------------------------------------------------------|

where the 3N+1 (3N+2, 3N+3) rows correspond to the x (y, z) coordinates of the Nth atom and the 3N+1 (3N+2, 3N+3) columns do the same.

Next, the BDF calls the UniMoVib program for the calculation of frequencies and thermodynamic quantities. First are the results for the integrable representation
to which the vibration belongs, the vibrational frequency, the approximate mass, the force constants and the simple positive modes：

.. code-block:: 

     ************************************
     ***  Properties of Normal Modes  ***
     ************************************

     Results of vibrations:
     Normal frequencies (cm^-1), reduced masses (AMU), force constants (mDyn/A)

                                                       1                                 2                                 3
              Irreps                                  A1                                 E                                 E
         Frequencies                            733.9170                         1020.5018                         1021.2363
      Reduced masses                              7.2079                            1.1701                            1.1699
     Force constants                              2.2875                            0.7179                            0.7189
            Atom  ZA               X         Y         Z             X         Y         Z             X         Y         Z
               1   6        -0.21108  -0.57499  -0.00106      -0.04882   0.01679   0.10300       0.09664  -0.03546   0.05161
               2   1        -0.13918  -0.40351   0.04884      -0.06700  -0.59986  -0.13376      -0.37214  -0.36766  -0.03443
               3   1        -0.11370  -0.42014  -0.03047       0.26496   0.65294  -0.15254      -0.28591  -0.18743  -0.15504
               4   1        -0.19549  -0.38777  -0.01079       0.05490  -0.14087  -0.24770       0.15594   0.73490  -0.07808
               5  17         0.08533   0.23216   0.00014       0.00947  -0.00323  -0.01995      -0.01869   0.00699  -0.01000

Where each vibration mode is arranged in the order of vibration frequencies from smallest to largest, and the imaginary frequencies are ranked before all real
frequencies, so only the first few frequencies need to be checked to know the number of imaginary frequencies. Next, the thermochemical analysis results are printed：

.. code-block::

     *********************************************
     ***   Thermal Contributions to Energies   ***
     *********************************************

     Molecular mass            :        49.987388    AMU
     Electronic total energy   :      -499.841576    Hartree
     Scaling factor of Freq.   :         1.000000
     Tolerance of scaling      :         0.000000    cm^-1
     Rotational symmetry number:         3
     The C3v  point group is used to calculate rotational entropy.

     Principal axes and moments of inertia in atomic units:
                                         1                   2                   3
         Eigenvalues --                 11.700793          137.571621          137.571665
               X                         0.345094            0.938568           -0.000000
               Y                         0.938568           -0.345094           -0.000000
               Z                         0.000000            0.000000            1.000000

     Rotational temperatures             7.402388            0.629591            0.629591    Kelvin
     Rot. constants A, B, C              5.144924            0.437588            0.437588    cm^-1
                                       154.240933           13.118557           13.118553    GHz


     #   1    Temperature =       298.15000 Kelvin         Pressure =         1.00000 Atm
     ====================================================================================

     Thermal correction energies                              Hartree            kcal/mol
     Zero-point Energy                          :            0.037519           23.543449
     Thermal correction to Energy               :            0.040539           25.438450
     Thermal correction to Enthalpy             :            0.041483           26.030936
     Thermal correction to Gibbs Free Energy    :            0.014881            9.338203

     Sum of electronic and zero-point Energies  :         -499.804057
     Sum of electronic and thermal Energies     :         -499.801038
     Sum of electronic and thermal Enthalpies   :         -499.800093
     Sum of electronic and thermal Free Energies:         -499.826695
     ====================================================================================

The user can read the zero-point energy, enthalpy, Gibbs free energy, etc. as needed. Note that all of the above thermodynamic quantities are obtained under each of the following assumptions：

1. a frequency correction factor of 1.0；
2. a temperature of 298.15 K；
3. a pressure of 1 atm；
4. the simplicity of the electronic state is 1。

If the user's calculation does not fall under the above scenario, it can be specified by a series of keywords, such as the following, which represents a frequency correction factor of 0.98, a temperature of 373.15 K, a pressure of 2 atm, and a simplicity of 2 for the eletronic state. 

.. code-block:: bdf

    $bdfopt
    hess
     only
    scale
     0.98
    temp
     373.15
    press
     2.0
    ndeg
     2
    $end
    
Of particular note is the simplicity of the electronic state, which is equal to the spin multiplet (2S+1) for non-relativistic or scalar relativistic calculations
and where the electronic state does not have spatial simplicity; for electronic states with spatial simplicity, the spatial simplicity of the electronic state
should also be multiplied by the spatial simplicity of the electronic state, which is the number of dimensions of the incommensurable representation to which the
spatial part of the electronic wave function belongs. As for the relativistic calculations considering the spin-orbit coupling (e.g., TDDFT-SOC calculations),
the spin multiplicity should be replaced by the simplicity of the corresponding spin state (2J+1).

Sometimes, the frequency calculation is interrupted due to SCF non-convergence or other external reasons, when the calculation time can be saved by adding the ``restarthess`` keyword to the BDFOPT module for breakpoint continuation, e.g.

.. code-block:: bdf

    $bdfopt
    hess
     only
    restarthess
    $end

It is also worth noting that structural optimization and frequency analysis (the so-called opt+freq calculation) can be implemented sequentially in the same BDF
task, without the need to write two separate input files. For this purpose it is sufficient to change the input of the BDFOPT module to：

.. code-block:: bdf

    $bdfopt
    solver
     1
    hess
     final
    $end

where final means that the numerical Hessian calculation is performed only after the successful completion of the structural optimization; if the structural
optimization does not converge, the program simply quits with an error and does not perform the Hessian and frequency and thermodynamic quantities calculations.
If the structural optimization does not converge, the program quits without performing the Hessian, frequency, and thermodynamic quantities calculations.

Transition State Optimization: Transition State Optimization and Frequency Calculation for HCN/HNC Heterogeneous Reactions
-------------------------------------------------------------------------------------------------------------------------------

Prepare the following input file:

.. code-block:: bdf

    $compass
    title
       HCN <-> HNC transition state
    basis
       def2-SVP
    geometry
     C                  0.00000000    0.00000000    0.00000000
     N                  0.00000000    0.00000000    1.14838000
     H                  1.58536000    0.00000000    1.14838000
    end geometry
    $end

    $bdfopt
    solver
     1
    hess
     init+final
    iopt
     10
    $end

    $xuanyuan
    $end

    $scf
    rks
    dft
     b3lyp
    $end

    $resp
    geom
    $end

where ``iopt 10`` indicates the optimized transition state.

Whether optimizing the minima structure or the transition state, the program must generate an initial Hessian prior to the first structural optimization step for
use in subsequent structural optimization steps. In general, the initial Hessian should qualitatively match the exact Hessian under the initial structure, and in
particular, the number of imaginary frequencies must be the same. This requirement is easily satisfied for the optimization of very small value points, and even the
molecular mechanics level Hessian (the so-called "model Hessian") can be made to match the exact Hessian qualitatively, so the program uses the model Hessian as
the initial Hessian without calculating the exact Hessian. However, for transition state optimization, the model Hessian generally does not have an imaginary
frequency, so the exact Hessian must be generated as the initial Hessian. ``hess init+final`` in the above input file means that both the initial Hessian is generated
for the transition state optimization (this Hessian is not calculated on a structure with gradient 0), and the frequency and thermochemical quantities are
not calculated on a gradient 0 structure. The hess init+final in the above input file means that the initial Hessian is generated for the transition state optimization (this Hessian is not calculated on the structure with gradient 0,
the frequency and thermochemical quantities have no clear physical meaning, so only the Hessian is calculated without frequency analysis), and the Hessian is
calculated again after the structure optimization converges to obtain the frequency analysis results. It is also possible to replace ``init+final`` with ``init`` , i.e., to generate only the initial Hessian and
not to calculate the Hessian again after convergence of the structural optimization, but it is not recommended to omit the final keyword since transition state
optimization (and indeed all structural optimization tasks) generally requires checking the number of virtual frequencies of the final converged structure.


The computed output is similar to that of the optimized minimal value point structure. The final frequency analysis shows that the converged structure has
one and only one imaginary frequency（-1104 :math:`\rm cm^{-1}`）：

.. code-block:: 

     Results of vibrations:
     Normal frequencies (cm^-1), reduced masses (AMU), force constants (mDyn/A)

                                                       1                                 2                                 3
              Irreps                                  A'                                A'                                A'
         Frequencies                          -1104.1414                         2092.7239                         2631.2601
      Reduced masses                              1.1680                           11.9757                            1.0591
     Force constants                             -0.8389                           30.9012                            4.3205
            Atom  ZA               X         Y         Z             X         Y         Z             X         Y         Z
               1   6         0.04309   0.07860   0.00000       0.71560   0.09001   0.00000      -0.00274  -0.06631   0.00000
               2   7         0.03452  -0.06617   0.00000      -0.62958  -0.08802   0.00000       0.00688  -0.01481   0.00000
               3   1        -0.99304  -0.01621   0.00000       0.22954   0.15167   0.00000      -0.06313   0.99566   0.00000

This means that the transition state is indeed found.

In the above calculation, the theoretical level of the initial Hessian is the same as the theoretical level of the transition state optimization. Since the initial
Hessian only needs to be qualitatively correct, the initial Hessian can be calculated at a lower level and then the transition state can be optimized at a
higher theoretical level. Taking the above example, if we want to calculate the initial Hessian at the HF/STO-3G level and optimize the transition state at the B3LYP/Def2-SVP level, we can follow the following steps.


（1）Prepare the following input file named ``HCN-inithess.inp`` ：

.. code-block:: bdf

    $compass
    title
       HCN <-> HNC transition state, initial Hessian
    basis
       STO-3G
    geometry
     C                  0.00000000    0.00000000    0.00000000
     N                  0.00000000    0.00000000    1.14838000
     H                  1.58536000    0.00000000    1.14838000
    end geometry
    $end

    $bdfopt
    hess
     only
    $end

    $xuanyuan
    $end

    $scf
    rhf
    $end

    $resp
    geom
    $end

（2）Run the input file with BDF to obtain the Hessian file  ``HCN-inithess.hess`` ；

（3）Copy or rename  ``HCN-inithess.hess`` to ``HCN-optTS.hess`` ；

（4）Prepare the following input file, named ``HCN-optTS.inp``：

.. code-block:: bdf

    $compass
    title
       HCN <-> HNC transition state
    basis
       def2-SVP
    geometry
     C                  0.00000000    0.00000000    0.00000000
     N                  0.00000000    0.00000000    1.14838000
     H                  1.58536000    0.00000000    1.14838000
    end geometry
    $end

    $bdfopt
    solver
     1
    hess
     init+final
    iopt
     10
    readhess
    $end

    $xuanyuan
    $end

    $scf
    rks
    dft
     b3lyp
    $end

    $resp
    geom
    $end

Where the keyword ``readhess`` means to read a hess file with the same name as the input file (i.e. HCN-optTS.hess) as the initial Hessian.
Note that although this input file does not recalculate the initial Hessian, you still need to write ``hess init+final`` instead of ``hess final`` .

（5）Just run the input file.

Restricted Structural Optimization
-------------------------------------------------------

BDF also supports restricting the value of one or more internal coordinates in structure optimization by adding the constrain keyword to the BDFOPT module. the
first line after the constrain keyword is an integer (hereafter called N) indicating the total number of restrictions; lines 2 through N+1 define each
restriction. For example, the following input indicates the distance between the 2nd atom and the 5th atom (these two atoms do not necessarily need to be chemically
bonded to each other) to be restricted during structure optimization:

.. code-block:: bdf

    $bdfopt
    solver
     1
    constrain
     1
     2 5
    $end

The following input indicates that the distance between the 1st atom and the 2nd atom is restricted during structure optimization, and also the bond angles formed
by the 2nd, 5th and 10th atoms (again, no chemical bond is required between the 2nd and 5th atoms, or the 5th and 10th atoms)：

.. code-block:: bdf

    $bdfopt
    solver
     1
    constrain
     2
     1 2
     2 5 10
    $end
 
The following input indicates that the dihedral angles between the 5th, 10th, 15th, and 20th atoms are restricted during structure optimization, and also between the 10th, 15th, 20th, and 25th atoms：
 
.. code-block:: bdf

    $bdfopt
    solver
     1
    constrain
     2
     5 10 15 20
     10 15 20 25
    $end

Optimization of the conical intersection (CI) and the minimum energy intersection point (MECP)
---------------------------------------------------------------------------------------------------

The optimization of CIs and MECPs requires calling the DL-FIND external library, for which the following keywords are added to the input of the BDFOPT module.

.. code-block:: bdf

    solver
     0

Accordingly, ``solver 1`` in the previous examples means that the optimization isperformed using the BDF's own structural optimization code instead of DL-FIND. In
principle In principle, the optimization of minima and transition states can also be done with DL-FIND, but it is generally not as efficient as the BDF's own code, so DL-FIND should be called only for tasks that are not supported by the BDF's
own code, such as CI and MECP optimization.


The following is an example input for CI optimization, which computes the tapered intersection of the T1 and T2 states of the ethylene:

.. code-block:: bdf

    #----------------------------------------------------------------------
    # Gradient projection method for CI between T1 and T2 by TDDFT BHHLYP/6-31G
    #

    $COMPASS 
    Title
       C2H4 Molecule test run
    Basis
       6-31G
    Geometry
     C                  0.00107880   -0.00318153    1.43425054
     C                  0.00066030    0.00195132   -1.43437339
     H                  0.05960990   -0.89114967    0.84012371
     H                 -0.05830329    0.95445870    0.96064844
     H                  0.05950228    0.89180839   -0.84311032
     H                 -0.06267534   -0.95390169   -0.95768311
    END geometry
    nosym
    $END

    $bdfopt
    imulti             #Optimize CI
     2
    maxcycle           #Maximum number of optimization steps
     50
    tolgrad            #Convergence criterion for root mean square gradients
     1.d-4
    tolstep            #Convergence criterion for root mean square steps
     5.d-3
    $end

    $xuanyuan
    $end

    $SCF
    RKS
    charge
     0
    spinmulti
     1
    atomorb
    DFT
     BHHLYP
    $END

    $tddft
    imethod
     1
    isf
     1
    itda
     1
    nroot 
     5
    idiag
     1
    istore
     1
    crit_e
     1.d-8
    crit_vec
     1.d-6
    lefteig
    ialda
     4
    $end

    $resp
    geom
    norder
     1
    method
     2
    iroot
     1 
    nfiles
     1
    $end

    $resp
    geom
    norder
     1
    method
     2
    iroot
     2 
    nfiles
     1
    $end

    $resp
    iprt
     1
    QUAD
    FNAC
    double
    norder
     1
    method
     2
    nfiles
     1
    pairs
     1
     1 1 1 1 1 2
    $end

Note that this task requires not only the calculation of the gradients of the T1 and T2 states, but also the calculation of the non-adiabatic coupling vector
between the T1 and T2 states (done by the last RESP module), see tddft for the relevant keywords :doc:`tddft` , which are not repeated here. In the input of the BDFOPT module, ``imulti 2`` represents the optimization CI. similar to the normal structural
optimization task, the CI optimization outputs the gradient and step size convergence for each step, along with the energy convergence. For example, the output of the last optimization step of the above example is


.. code-block:: 

    Testing convergence  in cycle    6
        Energy  0.0000E+00 Target: 1.0000E-06 converged?  yes
      Max step  9.0855E-04 Target: 5.0000E-03 converged?  yes component     4
      RMS step  5.6602E-04 Target: 3.3333E-03 converged?  yes
      Max grad  5.5511E-05 Target: 1.0000E-04 converged?  yes component     1
      RMS grad  2.7645E-05 Target: 6.6667E-05 converged?  yes
    Converged!
     converged

Similar to the previous optimization tasks, the convergent CI structure is saved in In the ``.optgeom`` file, the coordinate unit is Bohr. Note that the value in the
row of energy is always displayed as 0, which does not mean that the system energy remains unchanged during CI optimization, but because the optimization CI will
not use the convergence of energy to judge whether it converges. For the same reason, the ``tolene`` keyword has no effect on CI optimization (and MECP optimization below).

The following is an example input file for optimizing MECP:

.. code-block:: bdf

    #----------------------------------------------------------------------
    # Gradient projection method for ISC between S0 and T1 by BHHLYP/6-31G
    #

    $COMPASS 
    Title
       C2H4 Molecule test run
    Basis
       6-31G
    Geometry
    C            -0.00000141      0.00000353      0.72393424
    C             0.00000417     -0.00000109     -0.72393515
    H             0.73780975     -0.54421247      1.29907106
    H            -0.73778145      0.54421417      1.29907329
    H             0.73777374      0.54421576     -1.29907129
    H            -0.73779427     -0.54423609     -1.29906321
    END geometry
    nosym
    $END

    $bdfopt
    imulti
     2
    maxcycle
     50
    tolgrad
     1.d-4
    tolstep
     5.d-3
    noncouple
    $end

    $xuanyuan
    $end

    $SCF
    RKS
    charge
     0
    spinmulti
     1
    atomorb
    DFT
    BHHLYP
    $END

    $resp
    geom
    norder
     1
    method
     1
    $end

    $SCF
    UKS
    charge
     0
    spinmulti
     3
    atomorb
    DFT
    BHHLYP
    $END

    $resp
    geom
    norder
     1
    method
     1
    $end

where the ``imulti 2`` and ``noncouple`` keywords are specified to perform MECP optimization. Note that the MECP optimization task requires the calculation of only two states (here S0 and T The output of the MECP optimization task is similar to the CI optimization task and is not described here.


Geometric Optimization Frequently Asked Questions
-------------------------------------------------------

False frequency problem
########################################################

Geometric optimization requires not only convergence of the structure (i.e., gradient and step size meet the convergence limits), but also the number of
imaginary frequencies of the resulting structure to meet the expected value, i.e., 0 when optimizing the structure of the minima, 1 when optimizing the transition
state, and higher order saddle points if the number of imaginary frequencies is greater than 1. When the actual number of virtual frequencies calculated does not
match the expected value, the structure needs to be adjusted and re-optimized.

 * When the actual calculated number of imaginary frequencies is less than the expected value, i.e., when the optimized transition state gets a structure with the number of imaginary frequencies of 0: this generally means that the obtained transition state structure is wrongly characterized, and the initial guess structure needs to be prepared again according to the common sense of chemistry. 
 * When the actual number of false frequencies is greater than the expected value, there are two possible cases：（1）the false frequencies are caused by the numerical error of the calculation, not the real existence. In this case, it can be solved by increasing the grid point, decreasing the integration truncation threshold, decreasing various convergence thresholds (such as SCF convergence threshold, structural optimization convergence threshold, etc.), etc.（2）The system does have a false frequency. In this case, we should check the simple positive mode corresponding to the false frequency from the output file, and perturb the converged structure along the direction of the simple positive mode, and then use the perturbed structure as the first guess to re-optimize the structure.
 * Note that it is impossible to determine whether a certain imaginary frequency is caused by numerical error from the frequency calculation results alone, but in general, the smaller the absolute value of the imaginary frequency, the more likely it is caused by numerical error, and vice versa, the more likely it is real.

Symmetry problem
########################################################

When the initial structure has a point group symmetry above group :math:`\rm C_1` , the structure
optimization may break the point group symmetry, e.g., when optimizing an ammonia
molecule with a planar structure with initial structure symmetry  :math:`\rm D_{3h}` , the structure optimization may result in a conical structure with symmetry :math:`\rm C_{3v}` .
By default the BDF forces the molecular point group symmetry to be maintained unless the system has a first order Jahn-Teller effect. If the user wants the BDF to break the
symmetry of the molecules, one of the following approaches can be taken：

 * Still optimize at high symmetry until convergence, and then calculate the frequencies. If false frequencies are present, perturb the molecular structure as in the previous subsection to eliminate them. If the molecule can be further reduced in energy by breaking the symmetry, then the perturbed molecular structure should be found to have reduced symmetry at this point, and the optimization should continue with that structure as the initial structure.
 * If a subgroup of the molecular point group is specified in the COMPASS module, the program will only keep the subgroup symmetry unbroken. If a  :math:`\rm C_1` group is specified, the program allows breaking the molecular symmetry in any way, maximizing the probability of obtaining a low-energy structure at the cost of not being able to use the point group symmetry to speed up the computation, resulting in increased computational effort.

Geometric optimization does not converge
########################################################

There are many factors that lead to the non-convergence of geometric optimization, including：

 * The presence of numerical noise in the energy, gradients;
 * The potential energy surface is too flat;
 * The molecule has more than one stable wave function, and the wave function jumps back and forth between the various stable solutions during structural optimization, and does not converge stably and consistently to the the same solution；
 * unreasonable molecular structure, e.g. wrong units of coordinates (e.g. the unit of coordinates is supposed to be Bohr, but the unit specified in the input file is Angstrom or vice versa), overdrawing or missing atoms, too close distances between non-bonded atoms, etc.

If the geometric optimization does not converge, or if there is no trend of convergence even though the maximum number of convergences has not been reached,
after repeatedly checking that the three-dimensional structure of the molecule is correct and reasonable, and that the wave function is not too close to the atom,
then the geometry of the molecule should be optimized. After repeatedly checking that the three-dimensional structure of the molecule is correct and reasonable,
and that the wave function converges normally, the following methods can be tried in turn:
 
 * Use the last frame of the task that does not converge as the initial structure and start the optimization again. In addition to manually copying the structure coordinates of the last frame into the input file, a In addition to manually copying the structural coordinates of the last frame into the input file, a simpler way is to add the ``restart`` keyword to the COMPASS module, e.g.
 
 
.. code-block:: bdf

    $compass
    title
     CH3Cl geomopt
    basis
     def2-SV(P)
    geometry
     C                  2.67184328    0.03549756   -3.40353093
     H                  2.05038141   -0.21545378   -2.56943947
     H                  2.80438882    1.09651909   -3.44309081
     H                  3.62454948   -0.43911916   -3.29403269
     Cl                 1.90897396   -0.51627638   -4.89053325
    end geometry
    restart
    $end

Suppose the input file is named ``CH3Cl-opt.inp`` ，then the program automatically reads the coordinates in ``CH3Cl-opt.optgeom`` as the initial structure at this point
(note that the program does not use the molecular coordinates in the ``geometry`` field at this point, but the molecular coordinates cannot be deleted). At first
glance, this may seem to be the same as simply increasing the maximum number of iterations for geometry optimization, but in fact it often works better than
simply increasing the maximum number of iterations, e.g., if the structure is reread after 100 steps of optimization and then re-optimized for 50 steps, the
convergence probability is often higher than if the structure is re-read for 150 consecutive steps. This is because the program regenerates the initial Hessian
when the structure is re-read to continue the optimization, thus avoiding the error accumulated by the quasi-Newton method of constructing the Hessian in multiple successive steps.


 * Decreasing the optimization step length, or trust radius. This is done by using the trust keyword, e.g.

.. code-block:: bdf

    $bdfopt
    solver
     1
    trust
     0.05
    $end
 
The default confidence radius is 0.3, so the new confidence radius should be less than 0.3. Note that the program will dynamically increase the confidence radius
if it detects that the confidence radius is too small. To avoid this behavior, the confidence radius can be set to a negative value, e.g.
  
.. code-block:: bdf

    $bdfopt
    solver
     1
    trust
     -0.05
    $end
  
To avoid this behavior, the confidence radius can be set to a negative value, e.g., the initial confidence radius is set to 0.05, and the confidence radius is
forbidden to exceed 0.05 during the entire structural optimization process.

 * For transition state optimization, the ``recalchess`` keyword can be used to specify that the exact Hessian is recalculated at several steps.

.. code-block:: bdf

    $bdfopt
    solver
     1
    iopt
     10
    hess
     init
    recalchess
     10
    $end

It indicates that the exact Hessian is recalculated every 10 steps of structural optimization, in addition to the exact Hessian calculated before structural optimization.

 * The lattice points are increased and the convergence thresholds of the integration truncation and SCF, etc., are decreased to reduce the numerical errors. Note that this method only works when the structural optimization is almost convergence but not full convergence.
