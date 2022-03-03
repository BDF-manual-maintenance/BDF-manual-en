Other techniques for self-consistent field calculations
=====================================

Initial Guesses for Self-Consistent Field Calculations
------------------------------------------------
The initial guess track of the self-consistent field calculation has a great impact on the convergence of the calculation. the BDF supports several initial guesses, as follows.

  * Atom : Combining molecular density matrix guesses using atomic density matrix, default option.
  * Huckel : semi-empirical Huckel method guess.
  * Hcore : diagonalized single-electron Hamiltonian guess.
  * Readmo : read in molecular orbitals as initial guess.
--  * Readdm : read in the density matrix as initial guess

BDF defaults to Atom guesses. The initial guess of the BDF can be changed in concise input mode using the keyword ``guess``, as follows

.. code-block:: bdf

    #! ch3cho.sh
    HF/6-31G guess=Hcore unit=Bohr
    
    geometry    # notice: unit in Bohr 
    C       0.1727682300       -0.0000045651       -0.8301598059
    C      -2.3763311896        0.0000001634        0.5600567139
    H       0.0151760290        0.0000088544       -2.9110013387
    H      -2.0873396672        0.0000037621        2.5902220967
    H      -3.4601725077       -1.6628370597        0.0320271859
    H      -3.4601679801        1.6628382651        0.0320205364
    O       2.2198078005        0.0000024315        0.2188182082
    end geometry

Here, we use the keyword ``guess=Hcore`` n the second line to specify the use of the Hcore guess. 18 iterations of the SCF converge.

.. code-block:: 

  Iter. idiis vshift  SCF Energy    DeltaE     RMSDeltaD    MaxDeltaD   Damping Times(S) 
   1    0   0.000 -130.488739529 174.680929376  0.401531162  5.325668770  0.0000   0.03
   2    1   0.000 -115.595786784  14.892952744  0.407402695  5.323804678  0.0000   0.02
   3    2   0.000 -126.823748834 -11.227962049  0.115300517  1.591646800  0.0000   0.03
   4    3   0.000 -150.870636785 -24.046887951  0.011394798  0.154813426  0.0000   0.02
   5    4   0.000 -151.121829169  -0.251192384  0.004498398  0.037875784  0.0000   0.03
   6    5   0.000 -150.900123989   0.221705180  0.008483436  0.119865266  0.0000   0.02
   7    6   0.000 -151.582006133  -0.681882144  0.011892345  0.122063906  0.0000   0.02
   8    7   0.000 -152.441656890  -0.859650757  0.007907887  0.062113717  0.0000   0.03
   9    8   0.000 -152.729229838  -0.287572947  0.003318529  0.037884676  0.0000   0.02
  10    2   0.000 -152.795374919  -0.066145081  0.005951772  0.054625652  0.0000   0.02
  11    3   0.000 -152.839276725  -0.043901806  0.000860488  0.010210210  0.0000   0.03
  12    4   0.000 -152.841131472  -0.001854746  0.000733951  0.007678730  0.0000   0.02
  13    5   0.000 -152.841752921  -0.000621449  0.000348937  0.003519950  0.0000   0.02
  14    6   0.000 -152.841816238  -0.000063316  0.000053288  0.000787592  0.0000   0.03
  15    7   0.000 -152.841819180  -0.000002942  0.000021206  0.000157533  0.0000   0.02
  16    8   0.000 -152.841819505  -0.000000325  0.000004796  0.000031694  0.0000   0.02
  17    2   0.000 -152.841819522  -0.000000016  0.000000698  0.000005497  0.0000   0.03
  18    3   0.000 -152.841819522  -0.000000000  0.000000236  0.000002276  0.0000   0.02
  diis/vshift is closed at iter =  18
  19    0   0.000 -152.8418195227 -0.000000000  0.000000078  0.000000848  0.0000   0.03

.. warning:: 
   The unit of numerator input for this example is Bohr, and the keyword ``unit=Bohr`` must be used to specify that the length of the coordinates is in ``Bohr`` 。

This example corresponds to the advanced input

.. code-block:: bdf

   $compass
   geometry
     C 0.1727682300 -0.0000045651 -0.8301598059
     C -2.3763311896 0.0000001634 0.5600567139
     H 0.0151760290 0.0000088544 -2.9110013387
     H -2.0873396672 0.0000037621 2.5902220967
     H -3.4601725077 -1.6628370597 0.0320271859
     H -3.4601679801 1.6628382651 0.0320205364
     O 2.2198078005 0.0000024315 0.2188182082
   end geometry
   unit # set unit of coordinates as Bohr
     bohr
   basis
     6-31g
   $end

   $xuanyuan
   $end

   $scf
   rhf
   guess # ask for hcore guess
     hcore
   $end

Reading in the initial guess orbitals
------------------------------------------------------------------------------------------
By default, the SCF calculation in BDF uses the atomic density matrix to construct the molecular density matrix to generate the initial guess orbitals. In practice, the user can read in the converged SCF molecular orbitals as the initial guess orbitals for the current SCF calculation. In this example, we first calculate a neutral :math:`\ce{H2O}` molecule and get the converged orbitals as the initial guess orbitals for the :math:`\ce{H2O+}` ions.

In the first step, the :math:`\ce{H2O}` molecule is calculated and the input file is prepared and named as ``h2o.inp``. The contents are as follows:

.. code-block:: bdf

    #!bdf.sh
    RKS/B3lyp/cc-pvdz     
    
    geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom 
    end geometry

After performing the calculation, the working directory generates the readable file ``h2o.scforb``, which holds the convergence orbits of the SCF calculation.


In the second step, the convergence orbit of the :math:`\ce{H2O}` molecule is used as an initial guess for the :math:`\ce{H2O+}` ion calculation, and the input file h2o+.inp is prepared, with the following contents.

.. code-block:: bdf

    #!bdf.sh
    ROKS/B3lyp/cc-pvdz guess=readmo charge=1
    
    geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom
    end geometry
    
    %cp $BDF_WORKDIR/h2o.scforb $BDF_TMPDIR/${BDFTASK}.inporb


这里，使用了关键词 ``guess=readmo`` ，指定要读入初始猜测轨道。初始猜测轨道是用 ``%`` 引导的拷贝命令从
环境变量 ``BDF_WORKDIR`` 定义的文件夹中的h2o.scforb文件复制为 ``BDF_TMPDIR`` 中的 ``${BDFTASK}.inporb`` 文件。
这里， ``BDF_WORKDIR`` 是执行计算任务的目录， ``BDF_TMPDIR`` 是BDF存储临时文件的目录。


Extending small basis group convergence orbits to large basis group initial guesses
------------------------------------------------
Initial guess orbits can be generated from different basis groups, again to accelerate computational convergence. This requires an extension of the initial guess track file. The track extensions should use the same base group, such as the cc-pVXZ series, ANO-RCC series, and other base groups. The orbital expansion currently supports only the advanced input mode.
For :math:`\ce{CH3CHO}` molecules, the orbitals are first calculated with cc-pVDZ and then expanded to the initial guess orbitals calculated with the cc-pVQZ basis set with the following inputs.

.. code-block:: bdf

    # First SCF calculation using small basis set cc-pvdz
    $compass
    geometry
    C       0.1727682300       -0.0000045651       -0.8301598059
    C      -2.3763311896        0.0000001634        0.5600567139
    H       0.0151760290        0.0000088544       -2.9110013387
    H      -2.0873396672        0.0000037621        2.5902220967
    H      -3.4601725077       -1.6628370597        0.0320271859
    H      -3.4601679801        1.6628382651        0.0320205364
    O       2.2198078005        0.0000024315        0.2188182082
    end geometry
    basis
     cc-pvdz
    unit # set unit of coordinates as Bohr
     Bohr
    $end
     
    $xuanyuan
    $end
     
    $scf
    rhf
    $end
    
    #change chkfil name into chkfil1
    %mv $BDF_WORKDIR/$BDFTASK.chkfil $BDF_WORKDIR/$BDFTASK.chkfil1
    
    $compass
    geometry
    C       0.1727682300       -0.0000045651       -0.8301598059
    C      -2.3763311896        0.0000001634        0.5600567139
    H       0.0151760290        0.0000088544       -2.9110013387
    H      -2.0873396672        0.0000037621        2.5902220967
    H      -3.4601725077       -1.6628370597        0.0320271859
    H      -3.4601679801        1.6628382651        0.0320205364
    O       2.2198078005        0.0000024315        0.2188182082
    end geometry
    basis
     cc-pvqz
    unit
     Bohr
    $end
    
    # change chkfil to chkfil1. notice, should use cp command since we will use
    # "$BDFTASK.chkfil" in the next calculation
    %cp $BDF_WORKDIR/$BDFTASK.chkfil $BDF_WORKDIR/$BDFTASK.chkfil2
    
    # copy converged SCF orbital as input orbital of the module expandmo
    %cp $BDF_WORKDIR/$BDFTASK.scforb $BDF_WORKDIR/$BDFTASK.inporb
    
    # Expand orbital to large basis set. The output file is $BDFTASK.exporb
    $expandmo
    overlap
    $end
     
    $xuanyuan
    $end
    
    # use expanded orbital as initial guess orbital
    %cp $BDF_WORKDIR/$BDFTASK.exporb $BDF_WORKDIR/$BDFTASK.scforb
    $scf
    RHF
    guess
     readmo
    iprtmo
     2
    $end

In the above input, the first RHF calculation is performed using the **cc-pVDZ** basis set, then the convergence track from the first SCF calculation is extended to the **cc-pVQZ** basis set using the expandmo module, and finally ``guess=readmo`` is used as the initial guess track to be read into the SCF.

The output of the expandmo module is that

.. code-block:: 

    |******************************************************************************|
    
        Start running module expandmo
        Current time   2021-11-29  22:20:50
    
    |******************************************************************************|
     $expandmo                                                                                                                                                                                                                                                       
     overlap                                                                                                                                                                                                                                                         
     $end                                                                                                                                                                                                                                                            
     /Users/bsuo/check/bdf/bdfpro/ch3cho_exporb.chkfil1
     /Users/bsuo/check/bdf/bdfpro/ch3cho_exporb.chkfil2
     /Users/bsuo/check/bdf/bdfpro/ch3cho_exporb.inporb
      Expanding MO from small to large basis set or revise ...
    
     1 Small basis sets
    
     Number of  basis functions (NBF):      62
     Maxium NBF of shell :        6
    
     Number of basis functions of small basis sets:       62
    
     2 Large basis sets
    
     Number of  basis functions (NBF):     285
     Maxium NBF of shell :       15
    
      Overlap expanding :                     1
     Read guess orb
     Read orbital title:  TITLE - SCF Canonical Orbital
    nsbas_small  62
    nsbas_large 285
    ipsmall   1
    iplarge   1
      Overlap of dual basis ...
      Overlap of large basis ...
     Write expanded MO to scratch file ...
    |******************************************************************************|
    
        Total cpu     time:          0.42  S
        Total system  time:          0.02  S
        Total wall    time:          0.47  S
    
        Current time   2021-11-29  22:20:51
        End running module expandmo
    |******************************************************************************|

It can be seen that the small base group has 62 tracks and the large base group has 285 tracks. expandmo reads in the regular tracks for SCF convergence, extends them to the large base group and writes them to a temporary file.

The output of the second SCF calculation is that

.. code-block:: 

    /Users/bsuo/check/bdf/bdfpro/ch3cho_exporb.scforb
    Read guess orb:  nden=1  nreps= 1  norb=  285  lenmo=  81225
    Read orbital title:  TITLE - orthognal Expand CMO
    Orbitals initialization is completed.
 
    ........
  Iter. idiis vshift  SCF Energy    DeltaE     RMSDeltaD    MaxDeltaD   Damping Times(S)
   1    0   0.000 -152.952976892 122.547522034  0.002218985  0.246735859  0.0000  16.30
   2    1   0.000 -152.983462881  -0.030485988  0.000367245  0.026196100  0.0000  16.83
   3    2   0.000 -152.983976045  -0.000513164  0.000086429  0.006856831  0.0000  17.18
   4    3   0.000 -152.984012062  -0.000036016  0.000016763  0.001472939  0.0000  17.02
   5    4   0.000 -152.984019728  -0.000007666  0.000010400  0.001012788  0.0000  17.42
   6    5   0.000 -152.984021773  -0.000002045  0.000003396  0.000328178  0.0000  17.28
   7    6   0.000 -152.984022197  -0.000000423  0.000001082  0.000075914  0.0000  17.40
   8    7   0.000 -152.984022242  -0.000000044  0.000000154  0.000008645  0.0000  17.28
   9    8   0.000 -152.984022243  -0.000000001  0.000000066  0.000005087  0.0000  19.38
  diis/vshift is closed at iter =   9
  10    0   0.000 -152.984022243  -0.000000000  0.000000007  0.000000584  0.0000  18.95
    
      Label              CPU Time        SYS Time        Wall Time
     SCF iteration time:       517.800 S        0.733 S      175.617 S


.. _momMethod:

Calculation of excited states by the maximum occupation of molecular orbitals (mom) method
------------------------------------------------
The mom (maximum occupation method) is a ΔSCF method that can be used to calculate excited states.
                                    
.. code-block:: bdf

    #----------------------------------------------------------------------
    # 
    # mom method: J. Liu, Y. Zhang, and W. Liu, J. Chem. Theory Comput. 10, 2436 (2014).
    #
    # gs  = -169.86584128
    # ab  = -169.62226127
    # T   = -169.62483480
    # w(S)= 6.69eV
    #----------------------------------------------------------------------
    $COMPASS 
    Title
     mom
    Basis
     6-311++GPP
    Geometry
     C       0.000000    0.418626    0.000000
     H      -0.460595    1.426053    0.000000
     O       1.196516    0.242075    0.000000
     N      -0.936579   -0.568753    0.000000
     H      -0.634414   -1.530889    0.000000
     H      -1.921071   -0.362247    0.000000
    End geometry
    Check
    $END
    
    $XUANYUAN
    $END
    
    $SCF
    UKS
    DFT
    B3LYP
    alpha
      10 2
    beta
      10 2
    $END
    
    %cp ${BDFTASK}.scforb $BDF_TMPDIR/${BDFTASK}.inporb

    # delta scf with mom
    $SCF
    UKS
    DFT
    B3LYP
    guess
     readmo
    alpha
     10 2
    beta
     10 2
    ifpair
    hpalpha
     1
     10 0 
     11 0 
    iaufbau
     2
    $END
   
    # pure delta scf for triplet
    $SCF
    UKS
    DFT
    B3LYP
    alpha
      11 2
    beta
      9 2
    $END

This example performs three SCF calculations.

* For the first SCF, the ground state of the formamide molecule is calculated using the UKS method. The input specifies the occupancy of alpha and beta orbitals using the alpha and beta keywords, respectively. The base state of the formamide molecule is the singlet state S0, where the specified alpha and beta occupancies are the same. ``10 2`` The integrable designation indicates that A' and A" have 10 and 2 occupied orbitals, respectively. The SCF module will fill the orbitals with electrons according to the construction principle, from low to high orbital energy.
* For the second SCF, the S1 state of the formamide molecule is calculated using the UKS and mom methods. The key points here are: 1. the convergent orbitals read into the previous UKS step are specified using guess=readmo; 2. the occupation number of each symmetry orbital is set using alpha, beta keywords; 3. the variable ifpair is set, which needs to be used in conjunction with hpalpha, hpbeta to specify the hole-particle - HP) orbital pairs for electronic excitation; 4. The variable hpalpha is set to specify the HP orbital pairs for excitation. The number 1 indicates the excitation of a pair of HP orbitals, and the following two rows specify the orbital excitation. The first column indicates the excitation of electrons from the 10th alpha orbital to the 11th alpha orbital in the first integrable representation; the elements of the second column are all zero, indicating that no excitation is done for the orbital in the second integrable representation; 5. The iaufbau variable is set to 2, specifying that the mom calculation is to be performed.
* For the third SCF, the T1 state of the formamide molecule is calculated using the UKS method. In the input, we specify the orbital occupancies using the alpha and beta keywords, where the number of occupancies of the alpha orbital is ``11 2``, indicating 11 and 2 electrons occupying the alpha orbital with symmetries A' and A", respectively, and the occupancy of the beta orbital is ``9 2``. Since the required state for the solution is the lowest energy state for a given number of orbital occupancies, there is no need to specify iaufbau.

Here, the first SCF calculation converges to the result that

.. code-block:: 

     Superposition of atomic densities as initial guess.
     skipaocheck T F
     Solve HC=EC in pflmo space. F       12       75
     Initial guess energy =   -169.2529540680
    
     [scf_cycle_init_ecdenpot]
    Meomory for coulpotential         0.00  G
    
     Start SCF iteration......
    
    Iter. idiis vshift  SCF Energy    DeltaE     RMSDeltaD    MaxDeltaD   Damping Times(S)
     1    0   0.000 -169.411739263  -0.158785195  0.005700928  0.163822560  0.0000   0.20
    Turn on DFT calculation ...
     2    1   0.000 -169.743175119  -0.331435856  0.008905349  0.340815886  0.0000   0.42
     3    2   0.000 -169.232333660   0.510841459  0.006895796  0.296788710  0.0000   0.43
     4    3   0.000 -169.863405142  -0.631071482  0.000364999  0.015732911  0.0000   0.43
     5    4   0.000 -169.863345847   0.000059295  0.000209771  0.009205878  0.0000   0.42
     6    5   0.000 -169.865811301  -0.002465454  0.000027325  0.000606909  0.0000   0.43
     7    6   0.000 -169.865831953  -0.000020651  0.000008039  0.000357726  0.0000   0.43
     8    7   0.000 -169.865833199  -0.000001246  0.000003927  0.000114311  0.0000   0.42
     9    8   0.000 -169.865833401  -0.000000201  0.000000182  0.000004399  0.0000   0.43
    diis/vshift is closed at iter =   9
    10    0   0.000 -169.865833402  -0.000000000  0.000000139  0.000003885  0.0000   0.43
    
      Label              CPU Time        SYS Time        Wall Time
     SCF iteration time:         8.650 S        0.700 S        4.050 S
    
     Final DeltaE =  -4.4343551053316332E-010
     Final DeltaD =   1.3872600382452641E-007   5.0000000000000002E-005
    
     Final scf result
       E_tot =              -169.86583340
       E_ele =              -241.07729109
       E_nn  =                71.21145769
       E_1e  =              -371.80490197
       E_ne  =              -541.14538673
       E_kin =               169.34048477
       E_ee  =               148.48285541
       E_xc  =               -17.75524454
      Virial Theorem      2.003102

It can be seen that the first SCF calculation uses the atom guess and the energy of S0 is calculated to be -169.8658334023 a.u.. The second SCF calculation reads in the convergent orbitals of the first SCF and does the SCF calculation using the mom method, and the output file first indicates that the molecular orbitals were read in and gives the occupation

.. code-block::

      [Final occupation pattern: ]

   Irreps:        A'      A'' 
  
   detailed occupation for iden/irep:      1   1
      1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00
   detailed occupation for iden/irep:      1   2
      1.00 1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00
   Alpha      10.00    2.00

Here, the 10th alpha orbital of the ``A'``integrable representation is the occupied orbital and the 11th orbital is the empty orbital. The second SCF calculation reads in the converged orbitals of the first SCF and does the SCF calculation using the mom method, where the input asks to excite the electrons of the 10th orbital represented by ``A'`` to the 11th orbital. The output file first suggests that the molecular orbitals were read in and gives the occupation

.. code-block:: 

   Read initial orbitals from user specified file.
  
   /tmp/20117/mom_formamide.inporb
   Read guess orb:  nden=2  nreps= 2  norb=   87  lenmo=   4797
   Read orbital title:  TITLE - SCF Canonical Orbital
  
   Initial occupation pattern: iden=1  irep= 1  norb(irep)=   66
      1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 0.00
      1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00
  
  
   Initial occupation pattern: iden=1  irep= 2  norb(irep)=   21
      1.00 1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00
  
  
   Initial occupation pattern: iden=2  irep= 1  norb(irep)=   66
      1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00
  
  
   Initial occupation pattern: iden=2  irep= 2  norb(irep)=   21
      1.00 1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
      0.00
    
Here, iden=1 is the alpha orbital and irep=1 refers to the first integrable representation, and there are a total of norb=66 orbitals, where the occupation number of the 10th orbital is 0.00 and the occupation number of the 11th orbital is 1.00. After 14 SCF iterations, the converged S1 state energy is -169.6222628003 a.u., as follows.

.. code-block:: 

    Iter. idiis vshift  SCF Energy    DeltaE     RMSDeltaD    MaxDeltaD   Damping Times(S)
     1    0   0.000 -169.505632070 125.031578610  0.020428031  1.463174456  0.0000   0.45
     2    1   0.000 -169.034645773   0.470986296  0.036913522  1.562284831  0.0000   0.43
     3    2   0.000 -165.750862892   3.283782881  0.032162782  1.516480990  0.0000   0.43
     4    3   0.000 -169.560678610  -3.809815718  0.008588866  0.807859419  0.0000   0.43
     5    4   0.000 -169.596211021  -0.035532411  0.003887621  0.367391029  0.0000   0.42
     6    5   0.000 -169.620128518  -0.023917496  0.001826050  0.172456003  0.0000   0.43
     7    6   0.000 -169.621976725  -0.001848206  0.000486763  0.044630527  0.0000   0.43
     8    7   0.000 -169.622245116  -0.000268391  0.000113718  0.004980035  0.0000   0.43
     9    8   0.000 -169.622261269  -0.000016153  0.000112261  0.009715905  0.0000   0.42
    10    2   0.000 -169.622262553  -0.000001284  0.000043585  0.004092668  0.0000   0.42
    11    3   0.000 -169.622262723  -0.000000169  0.000031601  0.002792075  0.0000   0.42
    12    4   0.000 -169.622262790  -0.000000067  0.000010125  0.000848297  0.0000   0.43
    13    5   0.000 -169.622262798  -0.000000007  0.000003300  0.000273339  0.0000   0.43
     diis/vshift is closed at iter =  13
    14    0   0.000 -169.622262800  -0.000000002  0.000001150  0.000079378  0.0000   0.42
    
      Label              CPU Time        SYS Time        Wall Time
     SCF iteration time:        13.267 S        0.983 S        6.000 S
    
     Final DeltaE =  -1.8403909507469507E-009
     Final DeltaD =   1.1501625138328933E-006   5.0000000000000002E-005
    
     Final scf result
       E_tot =              -169.62226280
       E_ele =              -240.83372049
       E_nn  =                71.21145769
       E_1e  =              -368.54021347
       E_ne  =              -537.75897296
       E_kin =               169.21875949
       E_ee  =               145.28871749
       E_xc  =               -17.58222451
      Virial Theorem      2.002385
    
    
     [Final occupation pattern: ]
    
     Irreps:        A'      A'' 
    
     detailed occupation for iden/irep:      1   1
        1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 0.00
        1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00
    
After convergence of SCF, the orbital occupancy is printed again and it can be seen that the 10th orbital in the **alpha** orbital with ``A'``  integrable representation has no electron occupation and the 11th orbital has one electron occupation and the 11th orbital is occupied by one electron.

The third SCF calculation gives the **T1** state energy as -169.6248370697 a.u. The output is as follows：

.. code-block:: 

    Iter. idiis vshift  SCF Energy    DeltaE     RMSDeltaD    MaxDeltaD   Damping Times(S)
     1    0   0.000 -169.411739263  -0.158785195  0.083821477  9.141182225  0.0000   0.17
     Turn on DFT calculation ...
     2    1   0.000 -169.480549474  -0.068810211  0.066700318  6.978728919  0.0000   0.40
     3    2   0.000 -169.277735673   0.202813801  0.014778190  0.648183923  0.0000   0.42
     4    3   0.000 -169.613991196  -0.336255522  0.005923909  0.621843348  0.0000   0.42
     5    4   0.000 -169.620096778  -0.006105582  0.001967168  0.164506160  0.0000   0.40
     6    5   0.000 -169.623636999  -0.003540220  0.002722812  0.246425639  0.0000   0.42
     7    6   0.000 -169.624704514  -0.001067515  0.001064536  0.098138798  0.0000   0.42
     8    7   0.000 -169.624814882  -0.000110368  0.000525436  0.046392861  0.0000   0.42
     9    8   0.000 -169.624834520  -0.000019637  0.000179234  0.012966641  0.0000   0.42
    10    2   0.000 -169.624836694  -0.000002174  0.000063823  0.004902276  0.0000   0.42
    11    3   0.000 -169.624836922  -0.000000227  0.000017831  0.001440089  0.0000   0.43
    12    4   0.000 -169.624837025  -0.000000103  0.000034243  0.002618897  0.0000   0.42
    13    5   0.000 -169.624837065  -0.000000039  0.000006158  0.000466001  0.0000   0.40
    14    6   0.000 -169.624837068  -0.000000003  0.000003615  0.000354229  0.0000   0.42
    diis/vshift is closed at iter =  14
    15    0   0.000 -169.624837069  -0.000000001  0.000000966  0.000070404  0.0000   0.42
   
     Label              CPU Time        SYS Time        Wall Time
    SCF iteration time:        13.150 S        0.950 S        5.967 S
   
    Final DeltaE =  -1.1375220765330596E-009
    Final DeltaD =   9.6591808698539483E-007   5.0000000000000002E-005
   
    Final scf result
      E_tot =              -169.62483707
      E_ele =              -240.83629476
      E_nn  =                71.21145769
      E_1e  =              -368.57834907
      E_ne  =              -537.80483706
      E_kin =               169.22648799
      E_ee  =               145.32683246
      E_xc  =               -17.58477815
     Virial Theorem      2.002354

.. _SCFConvProblems:

Handling Non-Convergence of Self-Consistent Field Calculations
------------------------------------------------
When the SCF calculation is completed, the user must check whether the SCF has converged or not. The user must check the convergence of the SCF and only if it converges can the results of the SCF calculation (energy, Bourget analysis, orbital energy, etc.) be used and subsequent calculations performed. Note that the convergence of the SCF cannot be judged only by the presence or absence of errors at the end of the output file. Because even if the SCF does not converge, 
the program does not exit immediately, but only after the output of the SCF iterations and before the output of the SCF energy. indicates that

.. code-block::

    Warning !!! Total energy not converged!
    
And even in this case, the program still prints the energy, orbital information, and the results of the booster analysis after this information. Although these results cannot be used as official calculation results, they are useful for analyzing the reasons for the non-convergence of the SCF.

Common causes of SCF non-convergence include

 1. the HOMO-LUMO energy gap is too small, resulting in repeated changes in the occupation of the frontline orbitals. For example, two orbitals  :math:`\psi_1` and :math:`\psi_2`, at the Nth SCF iteration :math:`\psi_1` is the occupied orbitals and :math:`\psi_2` is the empty orbitals, 
 however, after constructing the Fock matrix based on such orbital occupancy and diagonalizing it, the orbitals of the N+1th SCF iteration are obtained, but the orbital energy of :math:`\psi_1` is higher than that of :math:`\psi_2`, so the electrons are transferred from :math:`\psi_1` orbitals to :math:`\psi_2` orbitals. But then, the Fock matrix of the N+1th SCF iteration changes a lot compared to the Nth SCF iteration, resulting in a lower orbital energy of :math:`\psi_1` than  :math:`\psi_2` in the N+2nd SCF iteration, so the orbital occupation number returns to the case of the Nth SCF iteration, and thus the orbital occupation number of the SCF iteration always changes and never converges.
 This situation is typified by the fact that the SCF energy alternately oscillates between two energies (or oscillates irregularly within a certain range) with an oscillation amplitude around :math:`10^{-4} \sim 1` Hartree, and the orbital occupation number printed at the end of the SCF is not as expected.
 2. The HOMO-LUMO energy gap is small, and although the orbital occupation number does not change for each step of the iteration, the orbital shape changes repeatedly, leading to non-convergence of the SCF oscillation. The typical performance of this case is similar to the previous one, but the amplitude of the oscillation is generally slightly smaller and the orbital occupation number printed at the end of the SCF is qualitatively consistent with the expectation.
 3. The numerical integration grid point is too small or the accuracy of the two-electron integration is too low, resulting in small oscillations of the SCF that do not converge due to numerical errors. This situation is typically characterized by irregular oscillations of the SCF energy with an amplitude below :math:`10^{-4}` Hartree, and the number of orbital occupations printed at the end of the SCF is qualitatively as expected.
 4. The basis group is nearly linearly correlated, or the projection of the basis group on the grid is nearly linearly correlated because the grid is too small. This situation is typically characterized by the SCF energy varying by more than 1 Hartree (not necessarily in oscillation, but also monotonically or essentially monotonically), the SCF energy being much lower than expected, and the number of orbit occupancies printed at the end of the SCF being completely unphysically realistic. When the SCF energy is very much lower than expected, the SCF energy may not even be displayed as a number, but as a string of asterisks.
 
以下是各类SCF不收敛问题的常见解决方法（一定程度上也适用于BDF以外的软件）：

 1. add an energy shift vshift, for both category 1 and category 2 cases, by adding to the $scf module of the input file.

.. code-block:: bdf

 vshift
  0.2

If significant oscillations are still observed, it shall gradually increase vshift until convergence. vshift tends to make the convergence of SCF monotonic, but setting vshift too large increases the number of iterations used to converge, so it is appropriate to increase maxiter when increasing vshift. When vshift increases to 1.0 and still fails to converge, one should Consider other methods.

 2. increase the density matrix damping damp for the class 2 case (it also has a slight effect on the class 1 case) by adding to the $scf module of the input file.
 
.. code-block:: bdf

 damp
  0.7

Note that damp can be used in conjunction with vshift, and the two effects are to some extent mutually reinforcing. If significant oscillations are still observed with damping set to 0.7, increase the damping while ensuring that it is less than 1. For example, next try 0.9, 0.95, etc. Similar to vshift, damp also tends to improve the monotonicity of SCF convergence, but too large a damp leads to slower convergence, so maxiter can be increased. when damp is increased to 0.99 and still fails to converge, other methods should be considered.

 3. turn off DIIS for cases 1 and 2, and when increasing vshift and damp does not converge, DIIS will speed up SCF convergence in most cases, but it may slow down or even prevent convergence when the HOMO-LUMO energy gap is particularly small. In the latter case, you can turn off DIIS by adding the NoDIIS keyword to the $scf module, increase maxiter, and set vshift and damp depending on the convergence.
 4. 关闭SMH，适用于第1类和第2类情况，且前3种方式都不奏效时，方法是在$scf模块里添加NoSMH关键词，增加maxiter，并视收敛情况设定vshift和damp。We have not yet encountered a situation where SMH does not converge and does not converge, but since SMH is a very new method for convergence of SCF, turning off SMH can be an alternative. However, since SMH is a very new method for convergence of SCF, we cannot rule out that SMH may have a negative impact on convergence in rare cases, so turning off SMH can be an alternative.
 5. Switch to the FLMO or iOI method for cases of type 1 and type 2, where the molecules are large (e.g. larger than 50 atoms) and where it is suspected that the SCF does not converge because the initial guessing accuracy of the atoms is too low or because of qualitative errors. See :ref:`the sections on FLMO and iOI methods <FLMOMethod.rst>` 。
 6. Calculate a similar system that converges more easily and then use the wave function of that system as an initial guess to converge the original system, for both type 1 and type 2 cases. For example, if the SCF calculation of a neutral dibasic transition metal complex does not converge, one can calculate the monovalent cation of its closed-shell layer and use the orbitals of the monovalent cation as the first guess for the SCF calculation of the neutral molecule after convergence (but note that since BDF does not yet support reading the RHF/RKS wave function as the first guess for the UHF/UKS calculation, the monovalent cation of the closed-shell layer should be calculated using UHF /UKS calculation). In extreme cases it is even possible to calculate the higher valence cations first, then add a small number (e.g. 2) of electrons to reconverge the SCF, then add a small number of subs, and so on until the original wave function of the neutral system is obtained. 
    Another common tool is to perform the SCF calculation under the small basis group first, and then use the :ref:`expandmo module <expandmo.rst>` to project the SCF orbitals of the small basis group onto the original basis group after convergence, and then iterate the SCF under the original basis group until convergence. 
 7. Increase the grid points, which is applicable to the case of type 3 and sometimes also valid for the case of type 4. This is done by using grid keywords, e.g.
 
.. code-block:: bdf

 grid
  fine

注意：（1）For meta-GGA generalized functions, the default grid point is already fine, so the grid point should be set to ultra fine;
（2）Increasing the grid point will increase the time spent in each SCF iteration step;
（3）Increasing the grid point will make the converged energy incomparable with other calculations without changing the grid, so if you want to compare this calculation with previous calculations, or to compare the energy/free energy obtained from this calculation with other calculations, etc., you must recalculate all relevant calculations already done with the same grid point as this input file. Therefore, if you want to compare this calculation with previous calculations, or if you want to compare the energy/free energy obtained from this calculation with the results of other calculations, etc., you must recalculate all the relevant calculations already done with the same grid points as this input file, even if those calculations already done can converge without increasing the grid points.
If the results do not improve after increasing the grid points, you should try other methods; if the results improve but still do not converge, you can further try to change the fine to ultra fine; if it still does not converge, you should consider the following methods.

 8. The threshold value for the double electron integration is set tightly for the category 3 case and sometimes for the category 4 case as well. This is done by adding to the SCF module.
 
.. code-block:: bdf

 optscreen
  1

This method, like increasing the grid point, also increases the time consumed for each SCF iteration and also leads to results that are not comparable to those without the optscreen. This method is only applicable to calculations without MPEC or MPEC+COSX enabled.

 9. set the threshold for determining the linear correlation of the base group loose, for the category 4 case. This is done by adding to the $scf module.
 
.. code-block:: bdf

 checklin
 tollin
  1.d-6

This method will make the converged energy incomparable with other calculations without changing the tollin. It is not recommended to set the tollin larger than 1.d-5, otherwise it will lead to larger errors. If the tollin is set to 1.d-5 and still there is a non-convergence of type 4, then the methods described above, such as increasing the grid point and changing the two-electron integration threshold, should be considered.

Note that among the above methods, if a method does not converge the SCF but makes it converge better than before, the next method should be tried with

.. code-block:: bdf

 guess
  readmo

The orbit of the last SCF iteration of the previous method is read as the first guess. However, if the previous method backfired and caused the SCF convergence to deteriorate, the next method should be tried either by starting again with an atomic guess or by picking the track of the last iteration of the other previously tried method as the first guess (this of course requires the user to back up the tracks obtained for each SCF convergence method in advance).

Acceleration algorithms for self-consistent field calculations
------------------------------------------------
An important feature of the BDF is the acceleration of the energy and gradient calculations of SCF and TDDFT using the **MPEC+COSX** method. Set up the MPEC+COSX calculation with the following inputs.

.. code-block:: bdf

    #! amylose2.sh
    HF/cc-pvdz  MPEC+COSX

    Geometry
    H      -5.27726610038004     0.15767995434597     1.36892178079618
    H      -3.89542800401751    -2.74423996083456    -2.30130324998720
    H      -3.40930212959730     3.04543096108345     1.73325487719318
    O      -4.25161610042910    -0.18429704053319     1.49882079466485
    H      -4.12153806480025     0.39113300040060    -0.47267019103680
    O      -3.93883902709049    -2.16385597983528    -1.37984323910654
    H      -3.65755506365314    -2.55190701717719     0.56784675873394
    H      -2.66688104102718    -3.13999999152083    -0.32869523309397
    O      -3.68737510690803     2.57255697808269     0.79063986197194
    H      -2.16845111442446     1.40439897322928     1.59675986910159
    H      -0.80004208156425     3.67692503357694    -0.87083105709857
    C      -3.47036908085237     0.21757398797107     0.38361581865084
    C      -3.08081604941874    -2.23618399620817    -0.25179522317288
    H      -1.85215308213129    -1.05270701067006     0.92020982572454
    C      -2.73634509645279     1.50748698767418     0.67208385967460
    O      -0.95388209186676     2.93603601652216    -0.08659407523165
    H      -2.34176605974133     2.08883703173396    -1.35500112054343
    C      -2.46637306624908    -0.89337899823852     0.07760781649778
    C      -1.77582007601201     1.83730601785282    -0.45887211416401
    O      -1.70216504605578    -0.48600696920536    -1.07005315975028
    H      -0.26347504436884     0.90841605388912    -1.67304510231922
    C      -0.87599906000257     0.65569503172715    -0.80788211986139
    H       1.05124197574425    -4.08129295376550    -0.80486617677089
    H       1.91283792081157     2.93924205088598    -0.71300301703422
    O       0.07007992244287     0.29718501862843     0.19143889205868
    H       1.28488995808993    -0.48228594245462    -1.27588009910221
    O       0.83243796215244    -3.05225096122844    -0.51820416035526
    H       0.03099092283770    -2.15700599981123     1.08682384153403
    H       0.99725792474852    -3.21082099855794     1.38542783977374
    O       1.92550793896406     1.99389906198042    -1.25576903593383
    H       2.32288890226196     1.52348902475463     0.72949896259198
    H       5.42304993860699     1.71940008598879    -1.13583497057179
    C       1.35508593454345    -0.11004196264200    -0.25348109013556
    C       0.98581793175676    -2.43946398581436     0.75228585517262
    H       1.91238990103301    -0.83125899736406     1.66788890655085
    C       2.32240292575108     1.05122704465611    -0.25278704698785
    O       4.65571492366175     1.63248206459704    -0.36643098789343
    H       3.77658595927138     0.23304608296485    -1.60079803407907
    C       1.86060292384221    -1.20698497780059     0.68314589788694
    C       3.72997793572998     0.57134806164321    -0.56599702816882
    O       3.14827793673614    -1.62888795836893     0.20457391544942
    H       5.12279093584136    -0.96659193933436     0.00181296891020
    C       4.14403492674986    -0.60389595307832     0.31494395641232
    O       4.31314989648861    -0.29843197973243     1.69336596603165
    H       3.37540288537848     0.07856300492440     2.10071295465512
    End geometry

If in advanced input mode, simply add the keyword ``MPEC+COSX`` to the COMPASS module input, e.g.

.. code-block:: bdf

    $compass
    Geometry
    H      -5.27726610038004     0.15767995434597     1.36892178079618
    H      -3.89542800401751    -2.74423996083456    -2.30130324998720
    H      -3.40930212959730     3.04543096108345     1.73325487719318
    O      -4.25161610042910    -0.18429704053319     1.49882079466485
    H      -4.12153806480025     0.39113300040060    -0.47267019103680
    O      -3.93883902709049    -2.16385597983528    -1.37984323910654
    H      -3.65755506365314    -2.55190701717719     0.56784675873394
    H      -2.66688104102718    -3.13999999152083    -0.32869523309397
    O      -3.68737510690803     2.57255697808269     0.79063986197194
    H      -2.16845111442446     1.40439897322928     1.59675986910159
    H      -0.80004208156425     3.67692503357694    -0.87083105709857
    C      -3.47036908085237     0.21757398797107     0.38361581865084
    C      -3.08081604941874    -2.23618399620817    -0.25179522317288
    H      -1.85215308213129    -1.05270701067006     0.92020982572454
    C      -2.73634509645279     1.50748698767418     0.67208385967460
    O      -0.95388209186676     2.93603601652216    -0.08659407523165
    H      -2.34176605974133     2.08883703173396    -1.35500112054343
    C      -2.46637306624908    -0.89337899823852     0.07760781649778
    C      -1.77582007601201     1.83730601785282    -0.45887211416401
    O      -1.70216504605578    -0.48600696920536    -1.07005315975028
    H      -0.26347504436884     0.90841605388912    -1.67304510231922
    C      -0.87599906000257     0.65569503172715    -0.80788211986139
    H       1.05124197574425    -4.08129295376550    -0.80486617677089
    H       1.91283792081157     2.93924205088598    -0.71300301703422
    O       0.07007992244287     0.29718501862843     0.19143889205868
    H       1.28488995808993    -0.48228594245462    -1.27588009910221
    O       0.83243796215244    -3.05225096122844    -0.51820416035526
    H       0.03099092283770    -2.15700599981123     1.08682384153403
    H       0.99725792474852    -3.21082099855794     1.38542783977374
    O       1.92550793896406     1.99389906198042    -1.25576903593383
    H       2.32288890226196     1.52348902475463     0.72949896259198
    H       5.42304993860699     1.71940008598879    -1.13583497057179
    C       1.35508593454345    -0.11004196264200    -0.25348109013556
    C       0.98581793175676    -2.43946398581436     0.75228585517262
    H       1.91238990103301    -0.83125899736406     1.66788890655085
    C       2.32240292575108     1.05122704465611    -0.25278704698785
    O       4.65571492366175     1.63248206459704    -0.36643098789343
    H       3.77658595927138     0.23304608296485    -1.60079803407907
    C       1.86060292384221    -1.20698497780059     0.68314589788694
    C       3.72997793572998     0.57134806164321    -0.56599702816882
    O       3.14827793673614    -1.62888795836893     0.20457391544942
    H       5.12279093584136    -0.96659193933436     0.00181296891020
    C       4.14403492674986    -0.60389595307832     0.31494395641232
    O       4.31314989648861    -0.29843197973243     1.69336596603165
    H       3.37540288537848     0.07856300492440     2.10071295465512
    End geometry
    Basis
      cc-pvdz
    MPEC+COSX # ask for the MPEC+COSX method
    $end

The **SCF** module will output a prompt about whether **MPEC+COSX** are both set to True.

.. code-block:: bdf

    --- PRINT: Information about SCF Calculation --- 
    ICTRL_FRAGSCF=  0
    IPRTMO=  1
    MAXITER=  100
    THRENE= 0.10E-07 THRDEN= 0.50E-05
    DAMP= 0.00 VSHIFT= 0.00
    IFDIIS= T
    THRDIIS= 0.10E+01
    MINDIIS=   2 MAXDIIS=   8
    iCHECK=  0
    iAUFBAU=  1
    INIGUESS=  0
    IfMPEC= T
    IfCOSX= T

Here, ``IfMPEC= T`` and ``IfCOSX= T`` indicates that the **MPEC+COSX** method is used to compute. the SCF iterative process is as follows.

.. code-block:: bdf

     [scf_cycle_init_ecdenpot]
    Meomory for coulpotential         0.02  G
    
     Start SCF iteration......
    
    
    Iter.   idiis  vshift       SCF Energy            DeltaE          RMSDeltaD          MaxDeltaD      Damping    Times(S) 
       1      0    0.000   -1299.6435521238     -23.7693069405       0.0062252375       0.2842668435    0.0000      2.69
       2      1    0.000   -1290.1030630508       9.5404890730       0.0025508000       0.1065204344    0.0000      1.65
       3      2    0.000   -1290.2258798561      -0.1228168053       0.0014087449       0.0742227520    0.0000      1.67
       4      3    0.000   -1290.4879683983      -0.2620885422       0.0002338141       0.0153879051    0.0000      1.64
       5      4    0.000   -1290.4955210658      -0.0075526675       0.0000713807       0.0049309441    0.0000      1.57
       6      5    0.000   -1290.4966349620      -0.0011138962       0.0000156009       0.0010663736    0.0000      1.51
       7      6    0.000   -1290.4966797420      -0.0000447800       0.0000043032       0.0002765334    0.0000      1.44
       8      7    0.000   -1290.4966810419      -0.0000012999       0.0000014324       0.0000978302    0.0000      1.37
       9      8    0.000   -1290.4966794202       0.0000016217       0.0000003030       0.0000173603    0.0000      1.40
      10      2    0.000   -1290.4966902283      -0.0000108081       0.0000000659       0.0000034730    0.0000      1.11
     diis/vshift is closed at iter =  10
      11      0    0.000   -1290.5003691464      -0.0036789181       0.0000225953       0.0009032949    0.0000      5.85
    
      Label              CPU Time        SYS Time        Wall Time
     SCF iteration time:       179.100 S        1.110 S       22.630 S
    
     Final DeltaE = -3.678918126752251E-003
     Final DeltaD =  2.259533940614071E-005  5.000000000000000E-005
     
     Final scf result
       E_tot =             -1290.50036915
       E_ele =             -3626.68312754
       E_nn  =              2336.18275840
       E_1e  =             -6428.96436179
       E_ne  =             -7717.90756825
       E_kin =              1288.94320647
       E_ee  =              2802.28123424
       E_xc  =                 0.00000000
      Virial Theorem      2.001208

On a desktop with an i9-9900K CPU, eight OpenMP threads compute in parallel in 22 seconds. The SCF calculation under the same conditions without MPEC+COSX method, the computation takes 110 seconds, and **MPEC+COSX** speeds up the computation by a factor of about **5**.
