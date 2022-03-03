This chapter introduces the basic use of the various functions of the BDF, and gives basic examples of calculations and analysis of data readings for specific calculation functions.

.. _FirstExample:

First example :math:`\ce{H2O}` RHF calculation for H2O molecules
================================================
Hartree-Fock is the most basic algorithm in quantum chemistry. In this subsection, we will guide the user through a BDF calculation and analyze the input and output information by using an example of Hartree-Fock calculation for a water molecule. Here, we first give the concise inputs to the BDF, and in order to understand the difference between the concise and advanced input modes of the BDF, we also give the advanced input file for each concise input.


Preparing Inputs
-------------------------------------------------------
First, prepare the input file for the Hartree-Fock calculation of the single-point energy of water molecules, named ``h2o.inp``, with the following input：

.. code-block:: bdf 

    #!bdf.sh
    HF/3-21G    
  
    Geometry
    O
    H  1  R1 
    H  1  R1  2 109.
  
    R1=1.0     # input bond length with the default unit angstrom
    End geometry

The input is interpreted as follows.：
 - The first line must start with ``#!`` followed by a string named ``bdf.sh``, this can be any letter and array of words into a string, can not contain special characters other than ``.`` in addition to the special characters. The first line is the system reservation line, the user can use this string to mark the calculation task.
 - The second line, ``HF/3-21G`` is the calculation parameter control line for the BDF. ``HF`` stands for Hartree-Fock, and ``3-21G`` specifies that the calculation uses the ``3-21G`` basis group. The key parameter control line can be multiple consecutive lines.
 - The third row is an empty line and can be ignored. It is entered here to distinguish between different inputs and to enhance the readability of the input, and is recommended to be kept by the user.
 - The fourth and tenth lines are ``Geometry`` and ``End geometry`` , respectively, marking the start and end of the molecular geometry input, and the default unit of coordinates is angstrom.
 - The fifth through ninth lines enter the structure of the water molecule in the internal coordinate mode. (See :ref:`Internal coordinate format for molecular structure input <Internal-Coord>` for details)

This simple input corresponds to the advanced BDF input as follows：

.. code-block:: bdf 

  $compass
  Geometry
    O
    H 1 1.0
    H 1 1.0 2 109.
  End geometry
  Basis
    3-21g  # set basis set as 3-21g
  $end
  
  $xuanyuan
  $end
  
  $scf
  RHF       # Restrictive Hartree-Fock Method
  Charge    # The charge of the molecule is set to 0, and the default calculation is for neutral molecules with zero charge
    0    
  Spinmulti # spin-multiplicity 2S+1，the default calculation is for double electron system
    1    
  $end

As can be seen from the advanced input, BDF will execute the modules **COMPASS** ， **XUANYUAN** and **SCF** in order to complete the single-point energy calculation of the water molecule.
**COMPASS** is used to read in the basic information such as molecular structure, basis functions, etc., determine the symmetry of the molecule, rotate the molecule to the standard orientation (Standard orientation，see :ref:`the BDF use of group theory <Point-Group>`)，generate the symmetry-adapted orbitals, etc.,
and store such information into the ``h2o.chkfil`` 。 The keywords in **COMPASS** are

 * The molecular structure defined between ``Geometry`` and ``End geometry``;
 * ``Basis`` defines the basis group as ``3-21G``;

After executing the **COMPASS** module, BDF uses the **XUANYUAN** module to calculate the single and double electron integrals. Since the BDF defaults to the SCF method of **repeated calculation of double electron integrals**, i.e. **Integral Direct SCF** 。

Finally, the BDF executes the **SCF** module to complete the Hartree-Fock based self-consistent field calculation.

 * The ``RHF`` specifies the use of the restricted Hartree-Fock method;
 * ``Charge`` specifies that the charge of the system is 0;
 * ``Spinmulti`` specifies that the spin multi of the system is 1.

Here ``RHF`` is a mandatory keyword, and ``Charge`` and ``Spinmulti`` can be ignored for the restricted method.

Performing the calculation
-------------------------------------------------------
To perform the calculation, a shell script named ``run.sh`` is prepared and placed in the directory where the input file ``h2o.inp`` is located. The contents are as follows.

.. code-block:: shell

    #!/bin/bash

    # Set the BDF installation directory 
    export BDFHOME=/home/bsuo/bdf-pkg-pro
    # Set the BDF temporary file storage directory
    export BDF_TMPDIR=/tmp/$RANDOM

    # Set the available heap memory to be unrestricted, which may be limited by system administration if computing in a supercomputing environment
    ulimit -s unlimitted
    # Set the available computation time to be unlimited, which may be limited by system administration if computing in a supercomputing environment
    ulimit -t unlimitted

    # Set the number of OpenMP parallel threads
    export OMP_NUM_THREADS=4
    # Set the OpenMP availale heap memory size
    export OMP_STACKSIZE=1024M

    # Perform BDF calculations, note that the default output is printed to standard output
    $BDFHOME/sbin/bdfdrv.py -r h2o.inp 

The above is a ``Bash Shell`` script that defines some basic environment variables and executes the calculation using ``$BDFHOME/sbin/bdfdrv.py``. The environment variables defined in the script are：

 * ``BDFHOME`` ariable specifies the directory where BDF is installed.
 * ``BDF_TMPDIR`` variable specifies the BDF runtime temporary file storage directory.
 * ``ulimit -s unlimitted`` sets the available stack area memory for the program to be unlimitted.
 * ``ulimit -t unlimitted`` sets the program execution time to be unlimited.
 * ``export OMP_NUM_THREADS=4`` sets the number of OpenMP threads available for parallel computation.
 * ``export OMP_STACKSIZE=1024M`` sets the available Stack area memory for OpenMP to be 1024 megabytes.

The command to perform the calculation is

.. code-block:: shell

    $ ./run.sh h2o.inp &>h2o.out&

Since BDF prints the default output to the standard output, we use the Linux redirect command here to redirect the standard output to the file ``h2o.out`` 。

Analysis of the calculation results
-------------------------------------------------------
After the computation, the files ``h2o.out`` , ``h2o.chkfil`` , ``h2o.scforb`` will be obtained.
 
 * ``h2o.out`` is a text file, user readable, storing the BDF output printing information.
 * ``h2o.chkfil`` is a binary file, not user readable, used to pass data between different modules of the BDF; ``h2o.chkfil`` is a binary file, not user readable, used to pass data between different modules of the BDF.
 * ``h2o.scforb`` is a text file, user-readable, storing information on molecular orbital factors, orbital energies, etc. for self-consistent iterations of ``scf``, mainly used for restarting or as initial guess orbits for other scf calculations.

If the input file is in BDF simple input mode, ``h2o.out`` will first give some basic user setup information,

.. code-block:: bdf 

  |================== BDF Control parameters ==================|
 
    1: Input BDF Keywords
      soc=None    scf=rhf    skeleton=True    xcfuntype=None    
      xcfun=None    direct=True    charge=0    hamilton=None    
      spinmulti=1    
   
    2: Basis sets
       ['3-21g']
   
    3: Wavefunction, Charges and spin multiplicity
      charge=0    nuclearcharge=10    spinmulti=1    
   
    5: Energy method
       scf
   
    7: Acceleration method
       ERI
   
    8: Potential energy surface method
       energy

  |============================================================|

Here, the

 * ``Input BDF Keywords`` gives some basic control parameters.
 * ``Basis set`` gives the basis set used for the calculation.
 * ``Wavefunction, Charges and spinmulti`` gives the system charges, total nuclear charges and spin multiplicity (2S+1).
 * ``Energy method`` gives the energy calculation method.
 * ``Accleration method`` gives the two-electron integral calculation acceleration method.
 * ``Potential energy surface method`` gives the potential energy surface calculation method, here it is a single point energy calculation.

Subsequently, the system executes the **COMPASS**module, which gives the following prompt：

.. code-block:: 
  
    |************************************************************|
    
        Start running module compass
        Current time   2021-11-18  11:26:28

    |************************************************************|


The Cartesian coordinates of the input molecular structure in **Bohr** are then printed, as well as details of the basis functions for each type of atom

.. code-block:: 

    |---------------------------------------------------------------------------------|
    
     Atom   Cartcoord(Bohr)               Charge Basis Auxbas Uatom Nstab Alink  Mass
      O     0.000000  0.000000  0.000000  8.00    1     0     0     0   E     15.9949
      H     1.889726  0.000000  0.000000  1.00    2     0     0     0   E      1.0073
      H    -0.615235  1.786771  0.000000  1.00    2     0     0     0   E      1.0073
    
    |----------------------------------------------------------------------------------|
    
      End of reading atomic basis sets ..
     Printing basis sets for checking ....
    
     Atomic label:  O   8
     Maximum L  1 6s3p ----> 3s2p NBF =   9
     #--->s function
          Exp Coef          Norm Coef       Con Coef
               322.037000   0.192063E+03    0.059239    0.000000    0.000000
                48.430800   0.463827E+02    0.351500    0.000000    0.000000
                10.420600   0.146533E+02    0.707658    0.000000    0.000000
                 7.402940   0.113388E+02    0.000000   -0.404454    0.000000
                 1.576200   0.355405E+01    0.000000    1.221562    0.000000
                 0.373684   0.120752E+01    0.000000    0.000000    1.000000
     #--->p function
          Exp Coef          Norm Coef       Con Coef
                 7.402940   0.356238E+02    0.244586    0.000000
                 1.576200   0.515227E+01    0.853955    0.000000
                 0.373684   0.852344E+00    0.000000    1.000000
    
    
     Atomic label:  H   1
     Maximum L  0 3s ----> 2s NBF =   2
     #--->s function
          Exp Coef          Norm Coef       Con Coef
                 5.447178   0.900832E+01    0.156285    0.000000
                 0.824547   0.218613E+01    0.904691    0.000000
                 0.183192   0.707447E+00    0.000000    1.000000

Subsequently, the molecular symmetry is automatically determined and the rotation to the standard orientation mode is decided according to the user settings.

.. code-block:: 

    Auto decide molecular point group! Rotate coordinates into standard orientation!
    Threshold= 0.10000E-08 0.10000E-11 0.10000E-03
    geomsort being called!
    gsym: C02V, noper=    4
    Exiting zgeomsort....
    Representation generated
    Binary group is observed ...
    Point group name C(2V)                       4
    User set point group as C(2V)   
     Largest Abelian Subgroup C(2V)                       4
     Representation generated
     C|2|V|                    2

    Symmetry check OK
    Molecule has been symmetrized
    Number of symmery unique centers:                     2
    |---------------------------------------------------------------------------------|
    
     Atom   Cartcoord(Bohr)               Charge Basis Auxbas Uatom Nstab Alink  Mass
      O     0.000000  0.000000  0.000000  8.00    1     0     0     0   E     15.9949
      H     1.889726  0.000000  0.000000  1.00    2     0     0     0   E      1.0073
      H    -0.615235  1.786771  0.000000  1.00    2     0     0     0   E      1.0073
    
    |----------------------------------------------------------------------------------|
    
     Atom   Cartcoord(Bohr)               Charge Basis Auxbas Uatom Nstab Alink  Mass
      O     0.000000 -0.000000  0.219474  8.00    1     0     0     0   E     15.9949
      H    -1.538455  0.000000 -0.877896  1.00    2     0     0     0   E      1.0073
      H     1.538455 -0.000000 -0.877896  1.00    2     0     0     0   E      1.0073
    
    |----------------------------------------------------------------------------------|

Careful users may have noticed that the coordinates of the water molecules here are different from the ones entered. Finally, **COMPASS** generates symmetry adapted orbital and gives the integrable representations to which the dipole and quadrupole moments belong, printing a multiplication table for the ``C(2v)`` point group, giving the total number of basis functions and the number of symmetry adapted orbital for each integrable representation.

.. code-block:: 

    Number of irreps:    4
    IRREP:   3   4   1
    DIMEN:   1   1   1
    
     Irreps of multipole moment operators ...
     Operator  Component    Irrep       Row
      Dipole       x           B1          1
      Dipole       y           B2          1
      Dipole       z           A1          1
      Quadpole     xx          A1          1
      Quadpole     xy          A2          1
      Quadpole     yy          A1          1
      Quadpole     xz          B1          1
      Quadpole     yz          B2          1
      Quadpole     zz          A1          1
    
     Generate symmetry adapted orbital ...
     Print Multab
      1  2  3  4
      2  1  4  3
      3  4  1  2
      4  3  2  1
    
    |--------------------------------------------------|
              Symmetry adapted orbital                   
    
      Total number of basis functions:      13      13
    
      Number of irreps:   4
      Irrep :   A1        A2        B1        B2      
      Norb  :      7         0         4         2
    |--------------------------------------------------|

Here, the ``C(2v)`` point group has 4 one-dimensional integrable representations, labeled ``A1, A2, B1, B2`` , with ``7, 0, 4, 2`` symmetrically matched orbitals, respectively.

.. attention::

    Different quantum chemistry software may use different molecular standard orientations, resulting in some molecular orbitals being labeled with different integrable representations in different programs.

Finally, the ``COMPASS`` calculation ends normally, giving the following output.

.. code-block:: 

    |******************************************************************************|

        Total cpu     time:          0.00  S
        Total system  time:          0.00  S
        Total wall    time:          0.02  S
    
        Current time   2021-11-18  11:26:28
        End running module compass
    |******************************************************************************|


.. note::

    For each module execution of BDF, there will be informaton about the start of the execution and the time printed after the end of the execution, so that it is convenient for the user to locate exactly which calculation module has made an error.


The second module executed in this example is **XUANYUAN**, which is mainly used to calculate single and double electron integrals. Here, the **XUANYUAN** module only calculates and stores single-electron integrals and special double-electron integrals that require pre-screening of the integrals. If not specified, the BDF defaults to the direct calculation of the double electron integral to construct the Fock matrix. If user write in ``compass`` module the key word :ref:`Saorb<compass.saorb>`，double electron integral will be calculated and stored. The output of the **XUANYUAN** module is relatively simple and does not require special attention. Here, we give the most critical output.

.. code-block:: 

    [aoint_1e]
      Calculating one electron integrals ...
      S T and V integrals ....
      Dipole and Quadupole integrals ....
      Finish calculating one electron integrals ...
    
     ---------------------------------------------------------------
      Timing to calculate 1-electronic integrals                                      
    
      CPU TIME(S)      SYSTEM TIME(S)     WALL TIME(S)
              0.017            0.000               0.000
     ---------------------------------------------------------------
    
     Finish calculating 1e integral ...
     Direct SCF required. Skip 2e integral!
     Set significant shell pairs!
    
     Number of significant pairs:        7
     Timing caluclate K2 integrals.
     CPU:       0.00 SYS:       0.00 WALL:       0.00
    
From the output we see that the single-electron overlap, kinetic and nuclear attraction integrals are computed, and also the dipole and quadrupole moment integrals are computed. The two-electron integral calculation is ignored because the input requires the default integration to be calculated directly by SCF (Direct SCF).

Finally, the BDF invokes the **SCF** module to perform the **RHF** self-consistent field calculation. Information of interest are:

.. code-block:: 

     Wave function information ...
     Total Nuclear charge    :      10
     Total electrons         :      10
     ECP-core electrons      :       0
     Spin multiplicity(2S+1) :       1
     Num. of alpha electrons :       5
     Num. of beta  electrons :       5

The nuclear charge number, the total electron number, the core electron number for the pseudopotential calculation, the spin multiplicity, and the alpha and beta electron numbers are given here, and the user should check that the electronic states are correct. 
Then, the ``scf`` module first calculates the atoms and generates the initial guess density matrix for the molecular calculations.

.. code-block:: 

     [ATOM SCF control]
      heff=                     0
     After initial atom grid ...
     Finish atom    1  O             -73.8654283850
     After initial atom grid ...
     Finish atom    2  H              -0.4961986360
    
     Superposition of atomic densities as initial guess.

checking for possible linear correlations in the treatment of the basis functions.

.. code-block:: 

     Check basis set linear dependence! Tolerance =   0.100000E-04

It then proceeds to the SCF iterations, where after 8 iterations of convergence the accelerated convergence methods such as **DIIS** and **Level shift** are turned off and the energies are recalculated.

.. code-block:: 

    Iter. idiis vshift  SCF Energy    DeltaE     RMSDeltaD    MaxDeltaD   Damping Times(S) 
    1    0   0.000  -75.465225043  -0.607399386  0.039410497  0.238219747  0.0000   0.00
    2    1   0.000  -75.535887715  -0.070662672  0.013896819  0.080831047  0.0000   0.00
    3    2   0.000  -75.574187153  -0.038299437  0.004423591  0.029016074  0.0000   0.00
    4    3   0.000  -75.583580885  -0.009393732  0.000961664  0.003782740  0.0000   0.00
    5    4   0.000  -75.583826898  -0.000246012  0.000146525  0.000871203  0.0000   0.00
    6    5   0.000  -75.583831666  -0.000004768  0.000012300  0.000073584  0.0000   0.00
    7    6   0.000  -75.583831694  -0.000000027  0.000001242  0.000007487  0.0000   0.00
    8    7   0.000  -75.583831694  -0.000000000  0.000000465  0.000002549  0.0000   0.00
    diis/vshift is closed at iter =   8
    9    0   0.000  -75.583831694  -0.000000000  0.000000046  0.000000221  0.0000   0.00
    
      Label              CPU Time        SYS Time        Wall Time
     SCF iteration time:         0.017 S        0.017 S        0.000 S

Finally, the energy contributions and the Viry ratios of the different terms are printed.

.. code-block:: 

     Final scf result
       E_tot =               -75.58383169
       E_ele =               -84.37566837
       E_nn  =                 8.79183668
       E_1e  =              -121.94337426
       E_ne  =              -197.24569473
       E_kin =                75.30232047
       E_ee  =                37.56770589
       E_xc  =                 0.00000000
      Virial Theorem      2.003738

According to the Virial Theorem, the absolute value of the total potential energy of the system is two times the kinetic energy of the electron for a non-relativistic system, where the Virial ratio is ``2.003738``. The energy of the system is：

 * ``E_tot`` is the total energy of the system, i.e., ``E_ele`` + ``E_nn`` ;
 * ``E_ele`` is the electron energy, i.e. ``E_1e`` + ``E_ee`` + ``E_xc`` ;
 * ``E_nn``  is the nuclear repulsion energy;
 * ``E_1e``  is the single electron energy, i.e. ``E_ne`` + ``E_kin`` ;
 * ``E_ne``  is the energy of attraction of the nucleus to the electron;
 * ``E_kin`` is the electron kinetic energy;
 * ``E_ee`` is the two-electron energy, including Coulomb repulsion and exchange energy.
 * ``E_xc`` is the exchange-related energy, which is not 0 for DFT calculation.

The output of the energy printout is the occupancy of the orbitals, the orbital energy, the HUMO-LOMO energy and the energy gap, as shown below.

.. code-block:: 

     [Final occupation pattern: ]
    
     Irreps:        A1      A2      B1      B2  
    
     detailed occupation for iden/irep:      1   1
        1.00 1.00 1.00 0.00 0.00 0.00 0.00
     detailed occupation for iden/irep:      1   3
        1.00 0.00 0.00 0.00
     detailed occupation for iden/irep:      1   4
        1.00 0.00
     Alpha       3.00    0.00    1.00    1.00
    
    
     [Orbital energies:]
    
     Energy of occ-orbs:    A1            3
        -20.43281195      -1.30394125      -0.52260024
     Energy of vir-orbs:    A1            4
          0.24980046       1.23122290       1.86913815       3.08082943
    
     Energy of occ-orbs:    B1            1
         -0.66958992
     Energy of vir-orbs:    B1            3
          0.34934415       1.19716413       2.03295437
    
     Energy of occ-orbs:    B2            1
          -0.47503768
     Energy of vir-orbs:    B2            1
           1.78424252
    
     Alpha   HOMO energy:      -0.47503768 au     -12.92643838 eV  Irrep: B2      
     Alpha   LUMO energy:       0.24980046 au       6.79741929 eV  Irrep: A1      
     HOMO-LUMO gap:       0.72483814 au      19.72385767 eV

Here

 * ``[Final occupation pattern:]``gives the orbital occupation. Since we are performing a restricted Hartree-Fock calculation, the occupation is given only for the Alpha orbit, which is given separately according to the integrable representation. From this example, it can be seen that the first 3 of the A1 orbitals and the 1st of the B1 and B2 orbitals are occupied by 1 electron each. Since this example is an RHF, the alpha and beta orbitals are the same, so A1 indicates 3 double-occupied orbitals, and B1 and B2 indicate 1 double-occupied orbital each.
 * ``[Orbital energies:]`` The orbital energies are given separately according to the integrable representation.
 * ``Alpha   HOMO energy:`` gives the HOMO orbital energy in units au and eV; the integrable representation to which the orbital belongs, in this case B2.
 * ``Alpha   LUMO energy:`` the LUMO orbital energy is given in units of au and eV; the integrable representation to which the orbital belongs, in this case A1.
 * ``HOMO-LUMO gap:`` gives the energy difference between the HOMO and LUMO orbitals.

In order to reduce the number of output lines, BDF does not print the orbital composition and molecular orbital coefficients by default, but only gives the partial orbital occupation and orbital energy information according to the integrable representation. Only partial orbital occupancies and orbital energy information are given according to the integrable representation categories, as follows.

.. code-block:: 

      Symmetry   1 A1
    
        Orbital          1          2          3          4          5          6
        Energy     -20.43281   -1.30394   -0.52260    0.24980    1.23122    1.86914
        Occ No.      2.00000    2.00000    2.00000    0.00000    0.00000    0.00000
    
    
      Symmetry   2 A2
    
    
      Symmetry   3 B1
    
        Orbital          8          9         10         11
        Energy      -0.66959    0.34934    1.19716    2.03295
        Occ No.      2.00000    0.00000    0.00000    0.00000
    
    
      Symmetry   4 B2
    
        Orbital         12         13
        Energy      -0.47504    1.78424
        Occ No.      2.00000    0.00000
             
The **SCF** module finally prints the results of Mulliken and Lowdin Bourdin analysis, with information on the dipole moments of the molecules.

.. code-block:: 

     [Mulliken Population Analysis]
      Atomic charges: 
         1O      -0.7232
         2H       0.3616
         3H       0.3616
         Sum:    -0.0000
    
     [Lowdin Population Analysis]
      Atomic charges: 
         1O      -0.4756
         2H       0.2378
         3H       0.2378
         Sum:    -0.0000
    
    
     [Dipole moment: Debye]
               X          Y          Z     
       Elec:-.1081E-64 0.4718E-32 -.2368E+01
       Nucl:0.0000E+00 0.0000E+00 0.5644E-15
       Totl:   -0.0000     0.0000    -2.3684

.. hint:: 
    1. add the ``iprtmo`` keyword to the input of the **SCF** module with a value of ``2`` to output detailed information about the molecular orbitals.
    2. add the ``molden`` keyword to the input of the **SCF** module to output the molecular orbitals and occupancies as a molden format file, which can be used by third-party programs for visualization or wave function analysis（such as `GabEdit <http://gabedit.sourceforge.net/>`_， `JMol <http://jmol.sourceforge.net>`_，
    `Molden <https://www.theochem.ru.nl/molden/>`_，`Multiwfn <http://sobereva.com/multiwfn/>`_），
    to calculate :ref:`wavefunction analysis <1e-prop>` ，or calculate :ref:`single electron property <1e-prop>` 。

