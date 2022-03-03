
.. _TD:

Time-dependent Density Generalized Function Theory
================================================

The BDF supports a variety of excited state calculation methods, including the linear response time-density generalized function (TDDFT) method based on the Kohn-Sham reference state, and the Tamm-Dancoff approximation (TDA) of the TDDFT method. Compared with other quantization software, the TDDFT module of BDF is unique.
The main features are

1. support for various spin-flip (spin-flip) methods.
2. support spin-matching TDDFT method X-TDDFT, which can effectively solve the problem of spin contamination of excited states when the reference state is an open-shell layer, and is suitable for excited state calculation of free radicals, transition metals and other systems.
3. support core excited state (core excited state) related calculations, such as the calculation of the X-ray absorption spectrum (XAS). The iVI method used in BDF can directly calculate all excited states in a higher energy interval without calculating the lower excited states, thus saving computational resources.
4. support the calculation of first-order non-adiabatic coupling matrix element (fo-NACME, or NACME for short), especially between excited states and excited states NACME is mainly used to study non-radiative leap processes, such as using Fermi's golden rule NACME is mainly used to study non-radiative leap processes, 
   such as the calculation of the endoconversion rate constant using Fermi's golden rule, or the study of endoconversion, photochemical processes using non-adiabatic kinetics, etc. Many quantum chemistry programs support NACME between ground and excited states, but fewer programs support NACME between excited and excited states. 
   Therefore, BDF has a unique advantage over most existing quantum chemistry programs for processes such as excited-to-excited state endoconversions and multi-state photochemical reactions.

In addition to TDDFT, BDF also supports the calculation of excited states at the SCF level using the :ref:`mom方法<momMethod>` .

.. danger::

    All general functions of the  **SCAN** family(e.g.,SCAN0，r2SCAN) suffer from the "triplet instability" problem :cite:`scan_problem`,
    Should not be used for SF-TDDFT calculations(e.g,triple excited states for closed-shell systems). TDA is recommended for this case. 


Closed-shell system calculations：R-TDDFT
----------------------------------------------------------

R-TDDFT is used to calculate closed-shell systems. If the ground state calculation starts from the RHF, the TDDFT module performs the TDHF calculation. 
To calculate the excitation energy of :math:`\ce{H2O}` molecules using TDDFT, a concise input is given as follows.

.. code-block:: bdf

  #!bdf.sh
  TDDFT/B3lyp/cc-pvdz iroot=1   
  
  geometry
  O
  H  1  R1
  H  1  R1  2 109.
  
  R1=1.0       # OH bond length in angstrom
  end geometry

Here, the keyword ``TDDFT/B3lyp/cc-pvdz`` specifies that the TDDFT calculation is performed, the generic function used is ``B3lyp`` , and the basis group is ``cc-pVDZ`` . 
The corresponding high-level input is

.. code-block:: bdf

  $compass
  Geometry
    O
    H 1 1.0
    H 1 1.0 2 109.
  End geometry
  Basis
    cc-pvdz
  $end
   
  $xuanyuan
  $end
   
  $scf
  RKS      # Restricted Kohn-sham
  DFT      # DFT exchange-correlation functional B3lyp
    b3lyp 
  $end
  
  # input for tddft
  $tddft
  iroot    # For each irrep, calculate 1 root. 
    1       #on default, 10 roots are calculated for each irreps if advanced input used
  $end


The four modules **COMPASS** , **XUANYUAN** , **SCF** and **TDDFT** are called sequentially to complete the computation. The **SCF** module performs the RKS calculation. Based on the results of the RKS calculation, the subsequent **TDDFT** calculation is performed.

Note that since the water molecule belongs to the :math:`\rm C_{2v}` point group, there are 4 integrable representations, and the excited states of different integrable representations are solved separately, so there are several ways to specify the number of excited states depending on the user's requirements, such as

（1）Calculate 1 excited state for each integrable representation.

.. code-block:: bdf
  
  $TDDFT
  iroot
   1
  $END

At this time, the excited state calculated by each irreducible representation has a high probability of being the excited state with the lowest energy under the irreducible representation, but this cannot be guaranteed, that is to say, there is a small probability that the excited state will converge to the second excited state or even higher. some excited state. If you want to increase the probability of getting the lowest excited state, you can write

.. code-block:: bdf
  
  $TDDFT
  iroot
   2
  $END

At this time, two excited states are calculated for each irreducible representation, and the probability that the first excited state calculated under each irreducible representation is the excited state with the lowest energy under the irreducible representation is higher than when iroot=1. 
In addition, at this time, the second excited state calculated under each irreducible representation has a high probability of being the excited state with the second lowest energy under the irreducible representation, but the probability of satisfying this point is higher than that of the first excited state calculated under the irreducible representation. 
The probability of being the excited state with the lowest energy under this irreducible representation is lower. If iroot is increased further, the calculated probability that the first excited state is the one with the lowest energy quickly approaches 100%, but never strictly reaches 100%.

For similar reasons, not only is it often necessary to set iroot greater than 1 when calculating 1 excited state, but when calculating N (N > 1) excited states, if you want to relatively reliably ensure that these N excited states are the lowest energy The N excited states of , also need to set iroot greater than N. 
In general, iroot should be set larger, for example, at least 3 larger than the desired number of excited states, when the molecule satisfies one of the following conditions: (1) the molecule has approximate point group symmetry; (2) the molecule Although it has exact point group symmetry, the calculation is performed at a lower point group due to program limitations or at the user's request, 
such as in the open-shell TDDFT (see below) calculation, because the open-shell TDDFT code does not support non- Abelian point group, and the calculation is performed under the largest Abelian subgroup instead. When the molecule does not belong to one of the above cases, the iroot only needs to be slightly larger than the desired number of excited states, eg 1-2 larger.

（2）Counting only one B1 excited state and one B2 excited state, and not counting the excited states in the other integrable representations.

.. code-block:: bdf

  #! tdtest.sh
  TDDFT/B3lyp/3-21G nroot=0,0,1,1
 
   Geometry
   ...
   End geometry

or

.. code-block:: bdf
  
  $TDDFT
  nroot
   0 0 1 1  # 也可输入为 0,0,1,1
  $END

where the nroot keyword indicates the number of excited states specified by the user for each integrable representation. Since the program internally arranges the integrable representations of the :math:`\rm C_{2v}` point group in the order of A1, A2, B1, and B2 (see the section on the ordering of the integrable representations of the point group), the above input indicates that only one excited state each of B1 and B2 is counted.

（3）Calculate the lowest 4 excited states without limiting the integrable representations of these excited states

.. code-block:: bdf

  #! tdtest.sh
  TDDFT/B3lyp/3-21G iroot=-4
 
   Geometry
   ...
   End geometry

or

.. code-block:: bdf
  
  $TDDFT
  iroot
   -4
  $END

In this case, the program uses the initial guessed excitation energy to determine how many excitation states should be solved for each integrable representation, but since the initial guessed order of excitation energy may be different from the fully converged excitation energy, the program cannot strictly guarantee that the 4 excitation states obtained must be the 4 lowest energy states. 
If the user requires a strict guarantee that the 4 excited states obtained are the lowest 4 excited states, the user should make the program calculate more than 4 excited states, e.g., 8 excited states, and then take the 4 lowest energy states.

The output of the Kohn-Sham calculation has already been described, so here we will only focus on the results of the **TDDFT** calculation. The program output will first give information about the settings of the TDDFT calculation, so that the user can easily check whether the settings are calculated or not, as follows.

.. code-block:: 

      --------------------------------------------------   
      --- PRINT: Information about TDDFT calculation ---   
      --------------------------------------------------   
   ERI Maxblk=     8M
   [print level]
    iprt= 0
   [method]
    R-TD-DFT 
    isf= 0
    SC Excitations 
    RPA: (A-B)(A+B)Z=w2*Z 
   [special choice for method]
    ialda= 0
   [active space]
    Full active space 
   [algorithm]
    Target Excited State in each rep / Diag method :
    1   A1       1   1
    2   A2       1   1
    3   B1       1   1
    4   B2       1   1
   [dvdson_parameters]
    iupdate =   3
    Nfac =  50
    Nmaxcycle=  50
    nblock   =  50
    crit_e   = 0.10E-06
    crit_vec = 0.10E-04
    crit_demo= 0.10E-07
    crit_indp= 0.10E-09
    guess    =  20
    dump     =   0
   [output eigenvector control]
    cthrd= 0.100
      -------------------------------------------------   
      --- END : Information about TDDFT calculation ---   
      -------------------------------------------------   

Here,

* ``R-TD-DFT`` indicates that the TDDFT based on the restricted basis wave function calculation is being performed.
* ``isf= 0`` indicates that no flipping spin is being computed.
* ``ialda= 0`` indicates that the ``Full non-collinear Kernel``is used, which is the default Kernel for the non-spin-flipping TDDFT.

The output below gives the number of roots computed for each non-collinear representation.

.. code-block:: 

    Target Excited State in each rep / Diag method :
    1   A1       1   1
    2   A2       1   1
    3   B1       1   1
    4   B2       1   1

The TDDFT module also prints information about occupied orbitals, virtual orbitals, and other active orbitals computed by TDDFT

.. code-block:: 

             Print [Active] Orbital List         
              ---[Alpha set]---
   idx irep (rep,ibas,type)       F_av(eV)     iact 
 ---------------------------------------------------
    1    1   A1     1   2          -520.34813    0.05
    2    1   A1     2   2           -26.42196    1.84
    3    3   B1     1   2           -13.66589    2.96
    4    1   A1     3   2            -9.50404    2.49
    5    4   B2     1   2            -7.62124    2.12
    6    1   A1     4   0             1.23186    9.86
    7    3   B1     2   0             3.27539   11.48
    8    3   B1     3   0            15.02893    7.40
    9    1   A1     5   0            15.44682    6.60
   10    1   A1     6   0            24.53525    4.35
   11    4   B2     2   0            25.07569    3.88
   12    3   B1     4   0            27.07545    6.17
   13    2   A2     1   0            33.09515    3.99
   14    1   A1     7   0            34.03695    5.08
   15    4   B2     3   0            39.36812    4.67
   16    3   B1     5   0            43.83066    4.86
   17    1   A1     8   0            43.91179    4.34
   18    3   B1     6   0            55.56126    4.35
   19    1   A1     9   0            56.13188    4.04
   20    4   B2     4   0            78.06511    2.06
   21    2   A2     2   0            80.16952    2.10
   22    1   A1    10   0            83.17934    2.38
   23    1   A1    11   0            94.37171    2.81
   24    3   B1     7   0            99.90789    2.86

Here, orbitals 1-5 are occupied orbitals and 6-24 are virtual orbitals, where the 5th and 6th orbitals are HOMO and LUMO orbitals, belonging to integrable representation B2 and integrable representation A1 respectively, with orbital energies of -7.62124 eV and 1.23186 eV respectively. 
Since the :math:`\ce{H2O}` molecule has 4 irreducible representations, the TDDFT solves for each integrable representation one by one. The system estimates the memory usage before proceeding to the Davidson iteration to solve the Casida equation.

.. code-block:: 

 ==============================================
  Jrep: 1  ExctSym:  A1  (convert to td-psym)
  Irep: 1  PairSym:  A1  GsSym:  A1
  Nexit:       1     Nsos:      33
 ==============================================
 Estimated memory for JK operator:          0.053 M
 Maxium memory to calculate JK operator:         512.000 M
 Allow to calculate    1 roots at one pass for RPA ...
 Allow to calculate    2 roots at one pass for TDA ...

  Nlarge=               33 Nlimdim=               33 Nfac=               50
  Estimated mem for dvdson storage (RPA) =           0.042 M          0.000 G
  Estimated mem for dvdson storage (TDA) =           0.017 M          0.000 G

Here, the system counts about 0.053 MB of memory needed to store the JK operator, and 512 MB of memory for the input setting (see ``memjkop`` keyword). 
The system suggests that the RPA calculation, i.e., the full TDDFT calculation can count 1 root per pass (one pass) and the TDA calculation can count 2 roots per pass. Since the molecular system is small, there is enough memory. 
For larger molecular systems, if the number of allowed roots per pass output here is less than the system setting, the TDDFT module will construct the JK operator by multiple integration calculations based on the maximum number of allowed roots, resulting in a decrease in computational efficiency and requiring the user to increase memory with the ``memjkop`` keyword.

Davidson iteration starts computing the output information as follows.

.. code-block:: 

      Iteration started !
  
     Niter=     1   Nlarge =      33   Nmv =       2
     Ndim =     2   Nlimdim=      33   Nres=      31
     Approximated Eigenvalue (i,w,diff/eV,diff/a.u.):
        1        9.5246226546        9.5246226546           0.350E+00
     No. of converged eigval:     0
     Norm of Residuals:
        1        0.0120867135        0.0549049429           0.121E-01           0.549E-01
     No. of converged eigvec:     0
     Max norm of residues   :  0.549E-01
     *** New Directions : sTDDFT-Davidson step ***
     Left  Nindp=    1
     Right Nindp=    1
     Total Nindp=    2
     [tddft_dvdson_ZYNI]
     Timing For TDDFT_AVmat, Total:         0.08s         0.02s         0.02s
                           MTrans1:         0.00s         0.02s         0.00s
                           COULPOT:         0.00s         0.00s         0.00s
                           AVint  :         0.08s         0.00s         0.02s
                           MTrans2:         0.00s         0.00s         0.00s

     TDDFT ZYNI-AV time-TOTAL         0.08 S         0.02 S         0.02 S 
     TDDFT ZYNI-AV time-Coulp         0.08 S         0.02 S         0.02 S 
     TDDFT ZYNI-AV time-JKcon         0.00 S         0.00 S         0.00 S 

         tddft JK operator time:         0.00 S         0.00 S         0.00 S 


     Niter=     2   Nlarge =      33   Nmv =       4
     Ndim =     4   Nlimdim=      33   Nres=      29
     Approximated Eigenvalue (i,w,diff/eV,diff/a.u.):
        1        9.3817966321        0.1428260225           0.525E-02
     No. of converged eigval:     0
     Norm of Residuals:
        1        0.0029082582        0.0074085379           0.291E-02           0.741E-02
     No. of converged eigvec:     0

The convergence information is as follows.

.. code-block:: 

       Niter=     5   Nlarge =      33   Nmv =      10
     Ndim =    10   Nlimdim=      33   Nres=      23
     Approximated Eigenvalue (i,w,diff/eV,diff/a.u.):
        1        9.3784431931        0.0000001957           0.719E-08
     No. of converged eigval:     1
     ### Cong: Eigenvalues have Converged ! ###
     Norm of Residuals:
        1        0.0000009432        0.0000023006           0.943E-06           0.230E-05
     No. of converged eigvec:     1
     Max norm of residues   :  0.230E-05
     ### Cong.  Residuals Converged ! ###

     ------------------------------------------------------------------
      Orthogonality check2 for iblock/dim =      0       1
      Averaged nHxProd =     10.000
      Ndim =        1  Maximum nonzero deviation from Iden = 0.333E-15
     ------------------------------------------------------------------

     ------------------------------------------------------------------
      Statistics for [dvdson_rpa_block]:
       No.  of blocks =        1
       Size of blocks =       50
       No.  of eigens =        1
       No.  of HxProd =       10      Averaged =    10.000
       Eigenvalues (a.u.) = 
            0.3446513056
     ------------------------------------------------------------------
  
The first line of the above output shows that the computation converges in 5 iterations. The system then prints the information of the converged electronic state.

.. code-block:: 

  No. 1  w=9.3784 eV  -76.0358398606 a.u.  f= 0.0767   D<Pab>= 0.0000   Ova= 0.5201
  CV(0):   A1( 3 )->  A1( 4 )  c_i:  0.9883  Per: 97.7%  IPA: 10.736 eV  Oai: 0.5163
  CV(0):   B1( 1 )->  B1( 2 )  c_i: -0.1265  Per:  1.6%  IPA: 16.941 eV  Oai: 0.6563
  Estimate memory in tddft_init mem:           0.001 M

The information in the first line is as follows

* ``No.     1    w=      9.3784 eV`` means that the first excited state has an excitation energy of ``9.3784 eV``;
* ``-76.0358398606 a.u.`` gives the total energy of the first excited state;
* ``f= 0.0767`` gives the intensity of the oscillator of the jump between the first excited state and the ground state;
* ``D<Pab>= 0.0000`` is the difference between the <S^2> value of the excited state and the <S^2> value of the ground state (for spin-conserving jumps, this value reflects the degree of spin contamination of the excited state; for spin-flip jumps, the difference between this value and the theoretical value ``S(S+1)(excited state) - S(S+1)(ground state)`` reflects the degree of spin contamination of the excited state).
* ``Ova= 0.5201`` is the absolute overlap integral, which takes values in the range [0,1], the closer the value is to 0, the more pronounced the charge transfer characteristics of the corresponding excited state, otherwise the more pronounced the localized excitation characteristics).

Lines 2 and 3 give information on the excited main group states

* ``CV(0):`` where CV(0) means that the excitation is a Core to Virtual orbital excitation and 0 means that it is a Singlet excitation;
* ``A1( 3 ) -> A1( 4 )`` gives the occupied-vacancy orbital pair for the electron leap from the 3rd orbital of A1 to the 4th orbital of A1, which is the HOMO-2 to LUMO excitation when combined with the above output orbital information.
* ``c_i: 0.9883`` means that the linear combination factor of this jump in the whole excited state is 0.9883;
* ``Per: 97.7%`` indicates that this excited state accounts for 97.7% of the total number of excited states.
* ``IPA: 10.736 eV`` represents the energy difference of 10.736 eV between the two orbitals involved in the jump.
* ``Oai: 0.5163`` means that if the excited state has only the contribution of this one leap, then the absolute overlap integral of this excited state is 0.5001. From this information, it is convenient to know which leaps are local excitations and which leaps are charge transfer excitations.

After all integrable representations have been solved, all excited states are summarized in order of higher or lower energy output.

.. code-block:: 

  No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

    1  B2    1  B2    7.1935 eV    172.36 nm   0.0188   0.0000  99.8%  CV(0):  B2(   1 )->  A1(   4 )   8.853 0.426    0.0000
    2  A2    1  A2    9.0191 eV    137.47 nm   0.0000   0.0000  99.8%  CV(0):  B2(   1 )->  B1(   2 )  10.897 0.356    1.8256
    3  A1    2  A1    9.3784 eV    132.20 nm   0.0767   0.0000  97.7%  CV(0):  A1(   3 )->  A1(   4 )  10.736 0.520    2.1850
    4  B1    1  B1   11.2755 eV    109.96 nm   0.0631   0.0000  98.0%  CV(0):  A1(   3 )->  B1(   2 )  12.779 0.473    4.0820

Subsequently, the transition dipole moments also printed.

.. code-block:: 

  *** Ground to excited state Transition electric dipole moments (Au) ***
    State          X           Y           Z          Osc.
       1      -0.0000      -0.3266       0.0000       0.0188       0.0188
       2       0.0000       0.0000       0.0000       0.0000       0.0000
       3       0.0000       0.0000       0.5777       0.0767       0.0767
       4       0.4778      -0.0000       0.0000       0.0631       0.0631   


Calculation of the open-shell layer system：U-TDDFT
----------------------------------------------------------
The open-shell layer system can be calculated using U-TDDFT, e.g. for :math:`\ce{H2O+}` ions, the UKS calculation is performed first, and then the U-TDDFT calculation is used to calculate the excited state. A typical input is that

.. code-block:: bdf

    #!bdf.sh
    TDDFT/B3lyp/cc-pvdz iroot=4 group=C(1) charge=1    
    
    geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom 
    end geometry

Here, the keyword

* ``iroot=4`` specifies that 4 roots are calculated for each integrable representation.
* ``charge=1`` specifies that the charge of the system is +1.
* ``group=C(1)`` specifies that the C1 point group calculation is forced.

The corresponding high-level input is.

.. code-block:: bdf

  $compass
  #Notice: The unit of molecular coordinate is angstrom
  geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom 
  end geometry
  basis
    cc-pVDZ 
  group
   C(1)  # Force to use C1 symmetry
  $end
   
  $xuanyuan
  $end
   
  $scf
  uks
  dft
   b3lyp
  charge
   1
  spinmulti
   2
  $end
   
  $tddft
  iroot
   4
  $end

A few details to note about this input are

* The ``compass`` module uses the keyword ``group`` to force the calculation to use the ``C(1)`` point group;
* The ``scf`` module sets the ``UKS`` calculation with ``charge`` of ``1`` and ``spinmulti`` (spin multi, 2S+1) = 2;
* The ``iroot`` of the ``tddft`` module is set to count 4 roots per integrable representation, and since C1 symmetry is used, the calculation gives the first four excited states of the cation of water.

The U-TDDFT calculation is performed as can be seen from the following output.

.. code-block:: 

    --------------------------------------------------   
    --- PRINT: Information about TDDFT calculation ---   
    --------------------------------------------------   
 ERI Maxblk=     8M
 [print level]
  iprt= 0
 [method]
  U-TD-DFT 
  isf= 0
  SC Excitations 
  RPA: (A-B)(A+B)Z=w2*Z 

The calculation summarizes the output of the 4 excited states as

.. code-block:: 

  No. Pair   ExSym   ExEnergies     Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1
 
    1   A    2   A    2.1960 eV        564.60 nm   0.0009   0.0024  99.4% CO(bb):   A(   4 )->   A(   5 )   5.955 0.626    0.0000
    2   A    3   A    6.3479 eV        195.31 nm   0.0000   0.0030  99.3% CO(bb):   A(   3 )->   A(   5 )   9.983 0.578    4.1520
    3   A    4   A   12.0991 eV        102.47 nm   0.0028   1.9312  65.8% CV(bb):   A(   4 )->   A(   6 )  14.637 0.493    9.9032
    4   A    5   A   13.3618 eV         92.79 nm   0.0174   0.0004  97.6% CV(aa):   A(   4 )->   A(   6 )  15.624 0.419   11.1659

The third excited state has a large ``D<S^2>`` value, indicating a spin contamination problem.

Open-shell system: X-TDDFT (also called SA-TDDFT)
----------------------------------------------------------
X-TDDFT is a spin-matched TDDFT method for calculating the open-shell layer system. 
The open-shell system U-TDDFT triplet state coupled to a double-occupied to imaginary orbital excited state (labeled CV(1) in BDF) suffers from spin contamination and thus its excitation energy is often underestimated. X-TDDFT can be used to solve this problem. Considering :math:`\ce{N2+}` molecules, the concise computational inputs to X-TDDFT are

.. code-block:: bdf

   #! N2+.sh
   X-TDDFT/b3lyp/aug-cc-pvtz group=D(2h) charge=1 spinmulti=2 iroot=5

   Geometry
     N 0.00  0.00  0.00
     N 0.00  0.00  1.1164 
   End geometry

Advanced input

.. code-block:: bdf

    $compass
    #Notice: The unit of molecular coordinate is angstrom
    Geometry
     N 0.00  0.00  0.00
     N 0.00  0.00  1.1164 
    End geometry
    basis
     aug-cc-pvtz
    group
     D(2h)  # Force to use D2h symmetry
    $end
     
    $xuanyuan
    $end
     
    $scf
    roks # ask for ROKS calculation
    dft
     b3lyp
    charge
     1
    spinmulti
     2
    $end
     
    $tddft
    iroot
     5
    $end

Here, the **SCF** module requires the ``ROKS`` method to calculate the ground state, and the **TDDFT** module will default to the **X-TDDFT** calculation.

The excited state output is.

.. code-block:: 

  No. Pair   ExSym   ExEnergies     Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1
 
    1 B2u    1 B2u    0.7902 eV       1569.00 nm   0.0017   0.0195  98.6%  CO(0): B2u(   1 )->  Ag(   3 )   3.812 0.605    0.0000
    2 B3u    1 B3u    0.7902 eV       1569.00 nm   0.0017   0.0195  98.6%  CO(0): B3u(   1 )->  Ag(   3 )   3.812 0.605    0.0000
    3 B1u    1 B1u    3.2165 eV        385.46 nm   0.0378   0.3137  82.6%  CO(0): B1u(   2 )->  Ag(   3 )   5.487 0.897    2.4263
    4 B1u    2 B1u    8.2479 eV        150.32 nm   0.0008   0.9514  48.9%  CV(1): B2u(   1 )-> B3g(   1 )  12.415 0.903    7.4577
    5  Au    1  Au    8.9450 eV        138.61 nm   0.0000   1.2618  49.1%  CV(0): B2u(   1 )-> B2g(   1 )  12.903 0.574    8.1548
    6  Au    2  Au    9.0519 eV        136.97 nm   0.0000   1.7806  40.1%  CV(1): B3u(   1 )-> B3g(   1 )  12.415 0.573    8.2617
    7 B1u    3 B1u    9.0519 eV        136.97 nm   0.0000   1.7806  40.1%  CV(1): B3u(   1 )-> B2g(   1 )  12.415 0.906    8.2617
    8 B2g    1 B2g    9.4442 eV        131.28 nm   0.0000   0.0061  99.0%  OV(0):  Ag(   3 )-> B2g(   1 )  12.174 0.683    8.6540
    9 B3g    1 B3g    9.4442 eV        131.28 nm   0.0000   0.0061  99.0%  OV(0):  Ag(   3 )-> B3g(   1 )  12.174 0.683    8.6540
   10  Au    3  Au    9.5281 eV        130.12 nm   0.0000   0.1268  37.0%  CV(0): B3u(   1 )-> B3g(   1 )  12.903 0.574    8.7379
   11 B1u    4 B1u    9.5281 eV        130.12 nm   0.0000   0.1267  37.0%  CV(0): B2u(   1 )-> B3g(   1 )  12.903 0.909    8.7379
   12  Au    4  Au   10.7557 eV        115.27 nm   0.0000   0.7378  49.1%  CV(1): B3u(   1 )-> B3g(   1 )  12.415 0.575    9.9655
   13 B3u    2 B3u   12.4087 eV         99.92 nm   0.0983   0.1371  70.4%  CV(0): B1u(   2 )-> B2g(   1 )  15.288 0.793   11.6185
   14 B2u    2 B2u   12.4087 eV         99.92 nm   0.0983   0.1371  70.4%  CV(0): B1u(   2 )-> B3g(   1 )  15.288 0.793   11.6185
   15 B1u    5 B1u   15.9005 eV         77.98 nm   0.7766   0.7768  32.1%  CV(0): B3u(   1 )-> B2g(   1 )  12.903 0.742   15.1103
   16 B2u    3 B2u   17.6494 eV         70.25 nm   0.1101   0.4841  92.0%  CV(0): B2u(   1 )->  Ag(   4 )  19.343 0.343   16.8592
   17 B3u    3 B3u   17.6494 eV         70.25 nm   0.1101   0.4841  92.0%  CV(0): B3u(   1 )->  Ag(   4 )  19.343 0.343   16.8592
   18  Ag    2  Ag   18.2820 eV         67.82 nm   0.0000   0.0132  85.2%  OV(0):  Ag(   3 )->  Ag(   4 )  19.677 0.382   17.4918
   19 B2u    4 B2u   18.5465 eV         66.85 nm   0.0021   1.5661  77.8%  CV(1): B2u(   1 )->  Ag(   4 )  19.825 0.401   17.7562
   20 B3u    4 B3u   18.5465 eV         66.85 nm   0.0021   1.5661  77.8%  CV(1): B3u(   1 )->  Ag(   4 )  19.825 0.401   17.7562
   21  Ag    3  Ag   18.7805 eV         66.02 nm   0.0000   0.2156  40.4%  CV(0): B3u(   1 )-> B3u(   2 )  20.243 0.337   17.9903
   22 B1g    1 B1g   18.7892 eV         65.99 nm   0.0000   0.2191  40.5%  CV(0): B2u(   1 )-> B3u(   2 )  20.243 0.213   17.9990
   23 B1g    2 B1g   18.8704 eV         65.70 nm   0.0000   0.2625  41.8%  CV(0): B3u(   1 )-> B2u(   2 )  20.243 0.213   18.0802
   24 B3g    2 B3g   18.9955 eV         65.27 nm   0.0000   0.2673  83.4%  CV(0): B2u(   1 )-> B1u(   3 )  20.290 0.230   18.2053
   25 B2g    2 B2g   18.9955 eV         65.27 nm   0.0000   0.2673  83.4%  CV(0): B3u(   1 )-> B1u(   3 )  20.290 0.230   18.2053
   26 B3u    5 B3u   19.0339 eV         65.14 nm   0.0168   1.6012  66.7%  CV(1): B1u(   2 )-> B2g(   1 )  20.612 0.715   18.2437
   27 B2u    5 B2u   19.0339 eV         65.14 nm   0.0168   1.6012  66.7%  CV(1): B1u(   2 )-> B3g(   1 )  20.612 0.715   18.2437
   28  Ag    4  Ag   19.0387 eV         65.12 nm   0.0000   0.0693  35.9%  CO(0):  Ag(   2 )->  Ag(   3 )  21.933 0.437   18.2484
   29  Ag    5  Ag   19.3341 eV         64.13 nm   0.0000   0.1694  44.7%  CO(0):  Ag(   2 )->  Ag(   3 )  21.933 0.457   18.5439
   30  Ag    6  Ag   19.8685 eV         62.40 nm   0.0000   1.7807  40.4%  CV(1): B3u(   1 )-> B3u(   2 )  21.084 0.338   19.0783
   31 B1g    3 B1g   19.8695 eV         62.40 nm   0.0000   1.7774  40.5%  CV(1): B2u(   1 )-> B3u(   2 )  21.084 0.213   19.0792
   32 B3g    3 B3g   19.9858 eV         62.04 nm   0.0000   1.6935  80.7%  CV(1): B2u(   1 )-> B1u(   3 )  21.038 0.231   19.1956
   33 B2g    3 B2g   19.9858 eV         62.04 nm   0.0000   1.6935  80.7%  CV(1): B3u(   1 )-> B1u(   3 )  21.038 0.231   19.1956
   34 B1g    4 B1g   19.9988 eV         62.00 nm   0.0000   1.7373  41.8%  CV(1): B3u(   1 )-> B2u(   2 )  21.084 0.213   19.2086
   35 B2g    4 B2g   20.2417 eV         61.25 nm   0.0000   0.2901  81.4%  CV(0): B1u(   2 )-> B3u(   2 )  22.628 0.228   19.4515
   36 B3g    4 B3g   20.2417 eV         61.25 nm   0.0000   0.2901  81.4%  CV(0): B1u(   2 )-> B2u(   2 )  22.628 0.228   19.4515
   37  Au    5  Au   21.2302 eV         58.40 nm   0.0000   0.2173  40.4%  CV(0): B2u(   1 )-> B2g(   2 )  22.471 0.157   20.4400
   38 B2g    5 B2g   22.1001 eV         56.10 nm   0.0000   0.0031  99.2%  OV(0):  Ag(   3 )-> B2g(   2 )  23.220 0.204   21.3099
   39 B3g    5 B3g   22.1001 eV         56.10 nm   0.0000   0.0031  99.2%  OV(0):  Ag(   3 )-> B3g(   2 )  23.220 0.204   21.3099
   40 B1g    5 B1g   23.4663 eV         52.84 nm   0.0000   0.0027  99.8%  OV(0):  Ag(   3 )-> B1g(   1 )  25.135 0.283   22.6761

Here, the 4th, 6th and 7th excited states are CV(1) states. Note that the ``D<S^2>`` values calculated by SA-TDDFT are based on the U-TDDFT formula, which gives an approximation of the spin contamination of the results if these states were calculated by U-TDDFT, but does not represent the actual spin contamination of these states, since SA-TDDFT guarantees that all excited states are strictly free of spin contamination. 
Therefore, a large value of ``D<S^2>`` for a state calculated by SA-TDDFT does not indicate that the result for that state is unreliable, but rather indicates that SA-TDDFT is an improvement over U-TDDFT for that state.

Calculation of the triplet excited state using the closed-shell singlet state as the reference state
------------------------------------------------------------------------------------------------------

Starting from the ground state of the closed-shell layer of the :math:`\ce{H2O}` molecule, the triplet excited state can be calculated. The simple input is.

.. code-block:: bdf

  #! bdf.sh
  tdft/b3lyp/cc-pvdz iroot=4 spinflip=1
  
  geometry
  O
  H  1  R1
  H  1  R1  2 109.
  
  R1=1.0     # OH bond length in angstrom
  end geometry

Note that although the keyword here is named spinflip, this calculation is not a spin-flip TDDFT calculation, since it calculates the :math:`M_S = 0` component of the triplet excited state instead of the :math:`M_S = 1` component. The corresponding high-level inputs are

.. code-block:: bdf

  $compass
  #Notice: Coordinate unit is angstrom
  geometry
  O
  H  1  R1
  H  1  R1  2 109.
  
  R1=1.0     # OH bond length in angstrom
  end geometry
  basis
   cc-pvdz
  group
   C(1)  # Force to use C1 symmetry
  $end
   
  $xuanyuan
  $end
   
  $scf
  rks    # ask for RKS calculation 
  dft
   b3lyp
  $end
   
  $tddft
  isf      # ask for triplet TDDFT calculation
   1 
  iroot
   4
  $end

When the TDDFT calculation is almost finished, there is an output message as follows.

.. code-block::

     *** List of excitations ***

  Ground-state spatial symmetry:   A
  Ground-state spin: Si=  0.0000

  Spin change: isf=  1
  D<S^2>_pure=  2.0000 for excited state (Sf=Si+1)
  D<S^2>_pure=  0.0000 for excited state (Sf=Si)

  Imaginary/complex excitation energies :   0 states
  Reversed sign excitation energies :   0 states

  No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

    1   A    1   A    6.4131 eV    193.33 nm   0.0000   2.0000  99.2%  CV(1):   A(   5 )->   A(   6 )   8.853 0.426    0.0000
    2   A    2   A    8.2309 eV    150.63 nm   0.0000   2.0000  97.7%  CV(1):   A(   4 )->   A(   6 )  10.736 0.519    1.8177
    3   A    3   A    8.4793 eV    146.22 nm   0.0000   2.0000  98.9%  CV(1):   A(   5 )->   A(   7 )  10.897 0.357    2.0661
    4   A    4   A   10.1315 eV    122.37 nm   0.0000   2.0000  92.8%  CV(1):   A(   4 )->   A(   7 )  12.779 0.479    3.7184

 *** Ground to excited state Transition electric dipole moments (Au) ***
    State          X           Y           Z          Osc.
       1       0.0000       0.0000       0.0000       0.0000       0.0000
       2       0.0000       0.0000       0.0000       0.0000       0.0000
       3       0.0000       0.0000       0.0000       0.0000       0.0000
       4       0.0000       0.0000       0.0000       0.0000       0.0000

where ``Spin change: isf=  1`` suggests that the calculation is for a state with a spin multiplet 2 larger than the ground state (i.e., a triplet state), and since the ground state is a single heavy state and the ground to excited state jump is spin-barred, the oscillator strength and jump dipole moment are both 0.

TDDFT **only calculates excited states with the same spin as the reference state by default**，For example, the ground state of an :math:`\ce{H2O}` molecule is a single heavy state, and the TDDFT value calculates the single heavy excited state; to calculate both the single heavy state and the triplet state, the input is 

.. code-block::

   #! H2OTDDFT.sh
   TDDFT/b3lyp/cc-pVDZ iroot=4 spinflip=0,1

   geometry
   O
   H   1  0.9
   H   1  0.9   2 109.0
   end geometry    

The system will run TDDFT twice to calculate the single heavy state and triplet state respectively, where the output of the single heavy state is

.. code-block::

     No. Pair   ExSym   ExEnergies     Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

    1  B2    1  B2    8.0968 eV        153.13 nm   0.0292   0.0000  99.9%  CV(0):  B2(   1 )->  A1(   4 )   9.705 0.415    0.0000
    2  A2    1  A2    9.9625 eV        124.45 nm   0.0000   0.0000  99.9%  CV(0):  B2(   1 )->  B1(   2 )  11.745 0.329    1.8656
    3  A1    2  A1   10.1059 eV        122.69 nm   0.0711   0.0000  99.1%  CV(0):  A1(   3 )->  A1(   4 )  11.578 0.442    2.0090
    4  B1    1  B1   12.0826 eV        102.61 nm   0.0421   0.0000  99.5%  CV(0):  A1(   3 )->  B1(   2 )  13.618 0.392    3.9857
    5  B1    2  B1   15.1845 eV         81.65 nm   0.2475   0.0000  99.5%  CV(0):  B1(   1 )->  A1(   4 )  16.602 0.519    7.0877
    6  A1    3  A1   17.9209 eV         69.18 nm   0.0843   0.0000  95.4%  CV(0):  B1(   1 )->  B1(   2 )  18.643 0.585    9.8240
    7  A2    2  A2   22.3252 eV         55.54 nm   0.0000   0.0000  99.8%  CV(0):  B2(   1 )->  B1(   3 )  24.716 0.418   14.2284
    ...

The output of the triplet state is

.. code-block::

    No. Pair   ExSym   ExEnergies     Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

    1  B2    1  B2    7.4183 eV        167.13 nm   0.0000   2.0000  99.4%  CV(1):  B2(   1 )->  A1(   4 )   9.705 0.415    0.0000
    2  A1    1  A1    9.3311 eV        132.87 nm   0.0000   2.0000  98.9%  CV(1):  A1(   3 )->  A1(   4 )  11.578 0.441    1.9128
    3  A2    1  A2    9.5545 eV        129.76 nm   0.0000   2.0000  99.2%  CV(1):  B2(   1 )->  B1(   2 )  11.745 0.330    2.1363
    4  B1    1  B1   11.3278 eV        109.45 nm   0.0000   2.0000  97.5%  CV(1):  A1(   3 )->  B1(   2 )  13.618 0.395    3.9095
    5  B1    2  B1   14.0894 eV         88.00 nm   0.0000   2.0000  97.8%  CV(1):  B1(   1 )->  A1(   4 )  16.602 0.520    6.6711
    6  A1    2  A1   15.8648 eV         78.15 nm   0.0000   2.0000  96.8%  CV(1):  B1(   1 )->  B1(   2 )  18.643 0.582    8.4465
    7  A2    2  A2   21.8438 eV         56.76 nm   0.0000   2.0000  99.5%  CV(1):  B2(   1 )->  B1(   3 )  24.716 0.418   14.4255
    ...

Since the single to triplet state jump is dipole-barred, the oscillator strength  ``f=0.0000``.

Spin-flip TDDFT calculation
----------------------------------------------------------

The BDF can calculate the triplet state not only from a single heavy state, but also from a **2S+1** heavy state with higher spin multiplicity (S = 1/2, 1, 3/2, ...), and upward spin-flip the **2S+3** heavy state; the spin-up **TDDFT/TDA** gives the alpha electron to unoccupied beta orbital lepton state of the double-occupied orbital, labeled as ``CV(1)`` excitation. 
Unlike the case where the ground state is a closed-shell single heavy state, the BDF calculation at this point is for the :math:`M_S = S+1` component of the **2S + 3** heavy state, so when the ground state is not a closed-shell single heavy state, the calculation can be called a spin-flip TDDFT calculation. 
The input file format for the spin-up TDDFT calculation is exactly the same as when the ground state is a closed-shell single heavy state and the triplet excited state is calculated, e.g., the following input file calculates the quadruplet excited state with the duplex state as the reference state.

.. code-block:: bdf

  ...
  $scf
  UKS
  ...
  spinmulti
   2
  $end
  
  $tddft
  ...
  isf
   1
  $end

In addition, the BDF can start from a triplet state and flip the spin down to calculate a single heavy state, which requires setting ``isf`` to ``-1``. Of course, it is also possible to flip down from a state with a higher spin multiplicity to calculate a state with a spin multiplicity of 2 less. 
Note that the spin-down **TDDFT/TDA** can only correctly describe electronic states that leap from the alpha orbital occupied by the open-shell layer to the beta orbital occupied by the open-shell layer, labeled as **OO(ab)** leap, while all other leap types of states have spin contamination problems.

Starting from the triplet state and reversing the spin downward to calculate the singlet state, the input is

.. code-block::

   #! H2OTDDFT.sh
   TDA/b3lyp/cc-pVDZ spinmulti=3 iroot=-4 spinflip=-1

   geometry
   O
   H   1  0.9
   H   1  0.9   2 109.0
   end geometry 

The output is

.. code-block::

      Imaginary/complex excitation energies :   0 states

  No. Pair   ExSym   ExEnergies     Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

    1   A    1   A   -8.6059 eV       -144.07 nm   0.0000  -1.9933  99.3% OO(ab):   A(   6 )->   A(   5 )  -6.123 0.408    0.0000
    2   A    2   A   -0.0311 eV     -39809.08 nm   0.0000  -0.0034  54.1% OO(ab):   A(   5 )->   A(   5 )   7.331 1.000    8.5747
    3   A    3   A    0.5166 eV       2399.85 nm   0.0000  -1.9935  54.0% OO(ab):   A(   6 )->   A(   6 )   2.712 0.999    9.1225
    4   A    4   A    2.3121 eV        536.24 nm   0.0000  -0.9994  99.9% OV(ab):   A(   6 )->   A(   7 )   4.671 0.872   10.9180

Here, the first three states are **OO(ab)** type excited states, where the first and the third states are basically pure singlet states (D is approximately equal to -2, i.e., the excited state is approximately equal to 0), and the second state is basically pure triplet state (D is approximately equal to 0); the fourth state is an **OV(ab)** type excited state with spin contamination problem (D is approximately equal to -1, i.e., the excited state is approximately equal to 1, between singlet and triplet states), and its excitation energy is not reliable.

.. warning::

   * BDF currently only supports spin-flipped TDA, but not spin-flipped TDDFT, but the calculation of triplet excited states with closed-shell singlet states as reference states is not subject to this restriction.


Calculation of UV-Vis and XAS spectra by iVI method
-------------------------------------------------------

The above examples are based on the TDDFT excited states solved by Davidson's method. In order to solve for an excited state with the Davidson method, it is generally necessary to solve for all excited states with lower energies than it. Therefore, when the energy of the target excited state is very high (e.g., when calculating the XAS spectrum), the Davidson method requires too many computational resources to obtain a result with limited computation time and memory. In addition, when using the Davidson method, the user must specify the number of excited states to be solved before the calculation, 
however, many times the user does not know the number of excited states he needs before the calculation, but only the approximate energy range of the excited states he needs, etc. This makes the user have to go through a series of trial and error, first set a small number of excited states for the calculation, and if he finds that If you find that you do not have the state you need, you can increase the number of excited states and recalculate until you find the state you need. Obviously, this will consume the user's energy and time for no reason.

BDF's iVI method provides a solution to this problem. In the iVI method, the user can specify the excitation energy range of interest (e.g., the entire visible region, or the K-edge region of carbon) without having to estimate how many excited states are in that range; the program can calculate all excited states in that range, without having to calculate excited states at energies lower than that range as in the Davidson method, on the one hand, and ensure that all excited states in that energy range are obtained On the other hand, it ensures that all excited states in the energy range are obtained without missing any. Two examples of calculations are given below.

（1）Calculation of the absorption spectrum of the DDQ radical anion in the 400-700 nm range（X-TDDFT，wB97X/LANL2DZ）

.. code-block:: bdf

  $COMPASS
  Title
   DDQ radical anion TDDFT
  Basis
   LANL2DZ
  Geometry # UB3LYP/def2-SVP geometry
   C                  0.00000000    2.81252550   -0.25536084
   C                  0.00000000    1.32952185   -2.58630187
   C                  0.00000000   -1.32952185   -2.58630187
   C                  0.00000000   -2.81252550   -0.25536084
   C                  0.00000000   -1.29206304    2.09336443
   C                 -0.00000000    1.29206304    2.09336443
   Cl                 0.00000000   -3.02272954    4.89063172
   Cl                -0.00000000    3.02272954    4.89063172
   C                  0.00000000   -2.72722649   -4.89578100
   C                 -0.00000000    2.72722649   -4.89578100
   N                  0.00000000   -3.86127688   -6.78015122
   N                 -0.00000000    3.86127688   -6.78015122  
   O                  0.00000000   -5.15052650   -0.22779097
   O                 -0.00000000    5.15052650   -0.22779097
  End geometry
  units
   bohr
  mpec+cosx # accelerate the calculation using MPEC+COSX
  $end

  $XUANYUAN
  rs
   0.3 # rs for wB97X
  $END

  $SCF
  roks
  dft
   wB97X
  charge
   -1
  $END

  $tddft
  iprt # print level
   2
  itda
   0
  idiag # selects the iVI method
   3
  iwindow
   400 700 nm # alternatively the unit can be given as au, eV or cm-1 instead of nm.
              # default is in eV if no unit is given
  itest
   1
  icorrect
   1
  memjkop
   2048
  $end

Since the molecule belongs to the :math:`\rm C_{2v}` point group, there are four integrable representations (A1, A2, B1, B2), and the program solves the TDDFT problem under each of the four integrable representations. Take A1 integrable representation as an example, after convergence of iVI iterations, the program outputs the following information.

.. code-block::

  Root 0, E= 0.1060649560, residual= 0.0002136455
  Root 1, E= 0.1827715245, residual= 0.0005375061
  Root 2, E= 0.1863919913, residual= 0.0006792424
  Root 3, E= 0.2039707800, residual= 0.0008796108
  Root 4, E= 0.2188244775, residual= 0.0015619745
  Root 5, E= 0.2299349293, residual= 0.0010684879
  Root 6, E= 0.2388141752, residual= 0.0618579646
  Root 7, E= 0.2609321083, residual= 0.0695001907
  Root 8, E= 0.2649984329, residual= 0.0759920121
  Root 9, E= 0.2657352154, residual= 0.0548521587
  Root 10, E= 0.2743644891, residual= 0.0655238098
  Root 11, E= 0.2766959875, residual= 0.0600950472
  Root 12, E= 0.2803090818, residual= 0.0587604503
  Root 13, E= 0.2958382984, residual= 0.0715968457
  Root 14, E= 0.3002756135, residual= 0.0607394762
  Root 15, E= 0.3069930238, residual= 0.0720773993
  Root 16, E= 0.3099721369, residual= 0.0956453409
  Root 17, E= 0.3141986951, residual= 0.0688103843
  Excitation energies of roots within the energy window (au):
  0.1060649560
   Timing Spin analyze :        0.01        0.00        0.00

   No.     1    w=      2.8862 eV     -594.3472248862 a.u.  f= 0.0000   D<Pab>= 0.0717   Ova= 0.5262
       CO(bb):   A1(  20 )->  A2(   4 )  c_i: -0.9623  Per: 92.6%  IPA:     8.586 eV  Oai: 0.5360
       CV(bb):   A1(  20 )->  A2(   5 )  c_i: -0.1121  Per:  1.3%  IPA:    11.748 eV  Oai: 0.3581
       CV(bb):   B1(  18 )->  B2(   6 )  c_i:  0.2040  Per:  4.2%  IPA:    13.866 eV  Oai: 0.4328

It can be seen that the program computes 17 excited states under this integrable representation, but only one of them (excitation energy 0.106 au = 2.89 eV) is within the user-specified wavelength interval (400-700 nm), and thus converges completely (in the form of small residuals); the rest of the excited states are known to be outside the user-interest range far before they converge, so the program does not try to converge anymore. The remaining excited states are known to be out of the user's range of interest well before they converge, and no further attempts are made to converge them (as indicated by the large residuals), thus saving much computational effort.

After all four integrable representations have been calculated, the program summarizes the results of each integrable representation as usual.

.. code-block::

    No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

      1  A1    2  A2    2.4184 eV    512.66 nm   0.1339   0.0280  93.0% OV(aa):  A2(   4 )->  A2(   5 )   7.064 0.781    0.0000
      2  B2    1  B1    2.7725 eV    447.19 nm   0.0000   0.0000  92.5% CO(bb):  B1(  18 )->  A2(   4 )   8.394 0.543    0.3541
      3  A2    1  A1    2.8862 eV    429.58 nm   0.0000   0.0000  92.6% CO(bb):  A1(  20 )->  A2(   4 )   8.586 0.526    0.4677
      4  B1    1  B2    3.0126 eV    411.55 nm   0.0000   0.0000  63.5% CO(bb):  B2(   4 )->  A2(   4 )   8.195 0.820    0.5942

（2）Calculation of carbon K-edge XAS spectra of ethylene（sf-X2C，M06-2X/uncontracted def2-TZVP）

.. code-block:: bdf

  $COMPASS
  Title
   iVI test
  Basis
   def2-TZVP
  geometry
   C -5.77123022 1.49913343 0.00000000
   H -5.23806647 0.57142851 0.00000000
   H -6.84123022 1.49913343 0.00000000
   C -5.09595591 2.67411072 0.00000000
   H -5.62911966 3.60181564 0.00000000
   H -4.02595591 2.67411072 0.00000000
  End geometry
  group
   c(1)
  uncontract # uncontract the basis set (beneficial for the accuracy of core excitations)
  $END

  $XUANYUAN
  heff
   3 # selects sf-X2C
  $END

  $SCF
  rks
  dft
   m062x
  $END

  $TDDFT
  imethod
   1 # R-TDDFT
  idiag
   3 # iVI
  iwindow
   275 285 # default unit: eV
  $end

The K-edge absorption of carbon is experimentally known to be around 280 eV, so the energy range here is chosen to be 275-285 eV. 15 excited states are calculated in this energy interval.

.. code-block::

    No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

      1   A    2   A  277.1304 eV      4.47 nm   0.0018   0.0000  97.1%  CV(0):   A(   5 )->   A(  93 ) 281.033 0.650    0.0000
      2   A    3   A  277.1998 eV      4.47 nm   0.0002   0.0000  96.0%  CV(0):   A(   6 )->   A(  94 ) 282.498 0.541    0.0694
      3   A    4   A  277.9273 eV      4.46 nm   0.0045   0.0000  92.8%  CV(0):   A(   7 )->   A(  94 ) 281.169 0.701    0.7969
      4   A    5   A  278.2593 eV      4.46 nm   0.0000   0.0000 100.0%  CV(0):   A(   8 )->   A(  95 ) 283.154 0.250    1.1289
      5   A    6   A  279.2552 eV      4.44 nm   0.0002   0.0000  85.5%  CV(0):   A(   4 )->   A(  93 ) 284.265 0.627    2.1247
      6   A    7   A  280.0107 eV      4.43 nm   0.0000   0.0000  96.6%  CV(0):   A(   8 )->   A(  96 ) 284.941 0.315    2.8803
      7   A    8   A  280.5671 eV      4.42 nm   0.0000   0.0000  97.0%  CV(0):   A(   5 )->   A(  94 ) 284.433 0.642    3.4366
      8   A    9   A  280.8642 eV      4.41 nm   0.1133   0.0000  93.3%  CV(0):   A(   2 )->   A(   9 ) 287.856 0.179    3.7337
      9   A   10   A  280.8973 eV      4.41 nm   0.0000   0.0000  90.1%  CV(0):   A(   1 )->   A(   9 ) 287.884 0.185    3.7668
     10   A   11   A  281.0807 eV      4.41 nm   0.0000   0.0000  66.8%  CV(0):   A(   6 )->   A(  95 ) 287.143 0.564    3.9502
     11   A   12   A  282.6241 eV      4.39 nm   0.0000   0.0000  97.7%  CV(0):   A(   7 )->   A(  95 ) 285.815 0.709    5.4937
     12   A   13   A  283.7528 eV      4.37 nm   0.0000   0.0000  65.1%  CV(0):   A(   4 )->   A(  94 ) 287.666 0.592    6.6223
     13   A   14   A  283.9776 eV      4.37 nm   0.0000   0.0000  92.1%  CV(0):   A(   6 )->   A(  96 ) 288.929 0.523    6.8471
     14   A   15   A  284.1224 eV      4.36 nm   0.0008   0.0000  98.2%  CV(0):   A(   7 )->   A(  96 ) 287.601 0.707    6.9920
     15   A   16   A  284.4174 eV      4.36 nm   0.0000   0.0000  93.7%  CV(0):   A(   3 )->   A(  93 ) 289.434 0.509    7.2869

However, it can be seen from the composition of the excited states that only the two excited states with excitation energies of 280.8642 eV and 280.8973 eV are excitations from the C 1s to the valence orbitals, while the rest of the excitations are excitations from the valence orbitals to the very high Rydberg orbitals, i.e., background absorption corresponding to valence electron ionization.

Plotting of absorption spectra with Gaussian widening
-------------------------------------------------------

The above calculations yield only the excitation energies and oscillator intensities for each excited state, while the user often needs to obtain the theoretically predicted peak shape of the absorption spectrum, which requires a Gaussian widening of the absorption of each excited state by a certain half-peak width. In BDF, this is done with the Python script plotspec.py 
(located under $BDFHOME/sbin/, where $BDFHOME is the BDF installation path). For example, suppose we have calculated the TDDFT excited state of the C60 molecule using BDF, and the corresponding output file is C60.out, then we can run

.. code-block:: bash

  $BDFHOME/sbin/plotspec.py C60.out

or

.. code-block:: bash

  $BDFHOME/sbin/plotspec.py C60

The script will output the following on the screen.

.. code-block::

  BDF output file: C60.out
  1 TDDFT output block(s) found
  Block 1: 10 excited state(s)
   - Singlet absorption spectrum, spin-allowed
  plotspec.py: exit successfully

and produces two files, one for C60.stick.csv, containing the absorption wavelengths and molar extinction coefficients for all excited states, which can be used to make stick graphs.

.. code-block::

  TDDFT Singlets 1,,
  Wavelength,Extinction coefficient,
  nm,L/(mol cm),
  342.867139,2899.779319,
  307.302300,31192.802393,
  237.635960,131840.430395,
  211.765024,295.895849,
  209.090150,134.498113,
  197.019205,179194.526059,
  178.561512,145.257962,
  176.943322,54837.570677,
  164.778366,548.752301,
  160.167663,780.089056,

The other is C60.spec.csv, which contains the absorption spectrum after Gaussian broadening (the default broadening FWHM is 0.5 eV).

.. code-block::

  TDDFT Singlets 1,,
  Wavelength,Extinction coefficient,
  nm,L/(mol cm),
  200.000000,162720.545118,
  201.000000,151036.824457,
  202.000000,137429.257570,
  ...
  998.000000,0.000000,
  999.000000,0.000000,
  1000.000000,0.000000,

These two files can be opened and plotted with Excel, Origin, and other plotting software.

The command line parameters can be used to control the plotting range, the Gaussian broadening FWHM, etc. Example.

.. code-block::

  # Plot the spectrum in the range 300-600 nm:
   $BDFHOME/sbin/plotspec.py wavelength=300-600nm filename.out

  # Plot an X-ray absorption spectrum in the range 200-210 eV,
  # using an FWHM of 1 eV:
   $BDFHOME/sbin/plotspec.py energy=200-210eV fwhm=1eV filename.out

  # Plot a UV-Vis spectrum in the range 10000 cm-1 to 40000 cm-1,
  # where the wavenumber is sampled at an interval of 50 cm-1:
   $BDFHOME/sbin/plotspec.py wavenumber=10000-40000cm-1 interval=50 filename.out

  # Plot an emission spectrum in the range 600-1200 nm, as would be
  # given by Kasha's rule (i.e. only the first excited state is considered),
  # where the wavelength is sampled at an interval of 5 nm:
   $BDFHOME/sbin/plotspec.py -emi wavelength=600-1200nm interval=5 filename.out

If you run $BDFHOME/sbin/plotspec.py without command line arguments, you can list all command line arguments and their usage, which will not be repeated here.

Excited State Structure Optimization
-------------------------------------------------------

BDF supports not only the calculation of TDDFT single point energies (i.e., excitation energies for a given molecular structure), but also structure optimization of excited states, numerical frequencies, etc. For this purpose, the ``$tddft`` module is required. For this purpose, a ``$resp`` module is added after the ``$tddft`` module to calculate the gradient of the TDDFT energy, and a ``$bdfopt`` module is added after the $compass module to use the TDDFT gradient information for structure optimization and frequency calculation (see  :ref:`结构优化与频率计算<GeomOptimization>` for details).

The following is an example of calculations to optimize the structure of the first excited state of butadiene at the B3LYP/cc-pVDZ level.

.. code-block:: bdf

  $COMPASS
  Title
   C4H6
  Basis
   CC-PVDZ
  Geometry # Coordinates in Angstrom. The structure has C(2h) symmetry
   C                 -1.85874726   -0.13257980    0.00000000
   H                 -1.95342119   -1.19838319    0.00000000
   H                 -2.73563916    0.48057645    0.00000000
   C                 -0.63203020    0.44338226    0.00000000
   H                 -0.53735627    1.50918564    0.00000000
   C                  0.63203020   -0.44338226    0.00000000
   H                  0.53735627   -1.50918564    0.00000000
   C                  1.85874726    0.13257980    0.00000000
   H                  1.95342119    1.19838319    0.00000000
   H                  2.73563916   -0.48057645    0.00000000
  End Geometry
  $END

  $BDFOPT
  solver
   1
  $END

  $XUANYUAN
  $END

  $SCF
  RKS
  dft
   B3lyp
  $END

  $TDDFT
  nroot
  # The ordering of irreps of the C(2h) group is: Ag, Au, Bg, Bu
  # Thus the following line specifies the calculation of the 1Bu state, which
  # happens to be the first excited state for this particular molecule.
   0 0 0 1
  istore
   1
  # TDDFT gradient requires tighter TDDFT convergence criteria than single-point
  # TDDFT calculations, thus we tighten the convergence criteria below.
  crit_vec
   1.d-6 # default 1.d-5
  crit_e
   1.d-8 # default 1.d-7
  $END

  $resp
  geom
  norder
   1 # first-order nuclear derivative
  method
   2 # TDDFT response properties
  nfiles
   1 # must be the same number as the number after the istore keyword in $TDDFT
  iroot
   1 # calculate the gradient of the first root. Can be omitted here since only
     # one root is calculated in the $TDDFT block
  $end

Note that in the above example, the meaning of the keyword ``iroot`` in the ``$resp`` module is different from the meaning of the keyword ``iroot`` in the ``$tddft`` module above. The former refers to calculating the gradient of the first excited state, and the latter refers to how many excited states are calculated for each irreducible representation.

After convergence of the structure optimization, the converged structure is output in the main output file.

.. code-block::

      Good Job, Geometry Optimization converged in     5 iterations!

     Molecular Cartesian Coordinates (X,Y,Z) in Angstrom :
        C          -1.92180514       0.07448476       0.00000000
        H          -2.21141426      -0.98128927       0.00000000
        H          -2.70870517       0.83126705       0.00000000
        C          -0.54269837       0.45145649       0.00000000
        H          -0.31040658       1.52367715       0.00000000
        C           0.54269837      -0.45145649       0.00000000
        H           0.31040658      -1.52367715       0.00000000
        C           1.92180514      -0.07448476       0.00000000
        H           2.21141426       0.98128927       0.00000000
        H           2.70870517      -0.83126705       0.00000000

                         Force-RMS    Force-Max     Step-RMS     Step-Max
      Conv. tolerance :  0.2000E-03   0.3000E-03   0.8000E-03   0.1200E-02
      Current values  :  0.5550E-04   0.1545E-03   0.3473E-03   0.1127E-02
      Geom. converge  :     Yes          Yes          Yes          Yes

In addition, the excitation energy in the excited state equilibrium structure can be read from the output of the last TDDFT module in the ``.out.tmp`` file, as well as the total energy of the excited state and the main components.

.. code-block::

   No.     1    w=      5.1695 eV     -155.6874121542 a.u.  f= 0.6576   D<Pab>= 0.0000   Ova= 0.8744
        CV(0):   Ag(   6 )->  Bu(  10 )  c_i:  0.1224  Per:  1.5%  IPA:    17.551 eV  Oai: 0.6168
        CV(0):   Bg(   1 )->  Au(   2 )  c_i: -0.9479  Per: 89.9%  IPA:     4.574 eV  Oai: 0.9035
        
  ...

    No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

      1  Bu    1  Bu    5.1695 eV    239.84 nm   0.6576   0.0000  89.9%  CV(0):  Bg(   1 )->  Au(   2 )   4.574 0.874    0.0000

The wavelength (240 nm) corresponding to the excitation energy in the excited state equilibrium structure is the fluorescence emission wavelength of butadiene.

Spin-orbit coupling calculation based on sf-X2C-TDDFT-SOC
----------------------------------------------------------

Relativistic effects include scalar relativity and spin-orbit coupling (SOC). The relativistic calculations require the use of **basis sets optimized for relativistic effects and the selection of a suitable Hamiltonian**. 
sf-X2C-TDDFT-SOC calculations are supported by BDF for all-electron sf-X2C, where sf-X2C refers to the scalar relativistic effects considered with spinless exact two-component (eXact Two-Component, X2C) Hamiltonians, and TDDFT-SOC refers to the spin-orbit coupling calculations based on TDDFT. TDDFT calculation of spin-orbit coupling. Note that although TDDFT is an excited state method, TDDFT-SOC can be used to calculate not only the contribution of SOC to the excited state energy and properties, but also the contribution of SOC to the ground state energy and properties.

Taking a molecule with a single heavy ground state as an example, the TDDFT calculation module needs to be called three times in sequence to complete the sf-X2C-TDDFT-SOC calculation. Among them, the first execution utilizes R-TDDFT and calculates the single heavy state, the second uses SF-TDDFT to calculate the triplet state, and the last reads in the wave functions of the first two TDDFT calculations and calculates the spin-orbit coupling of these states using the state interaction (SI) method. This can be clearly seen from the advanced input of the sf-X2C-TDDFT-SOC calculation for the :math:`\ce{CH2S}` molecule below.

.. code-block:: bdf

   $COMPASS
   Title
    ch2s
   Basis # Notice: we use relativistic basis set contracted by DKH2
     cc-pVDZ-DK 
   Geometry
   C       0.000000    0.000000   -1.039839
   S       0.000000    0.000000    0.593284
   H       0.000000    0.932612   -1.626759
   H       0.000000   -0.932612   -1.626759
   End geometry
   $END
   
   $xuanyuan
   heff  # ask for sf-X2C Hamiltonian
    3   
   hsoc  # set SOC integral as 1e+mf-2e
    2
   $end
   
   $scf
   RKS
   dft
     PBE0
   $end

   #1st: R-TDDFT, calculate singlets 
   $tddft
   isf
    0
   idiag
    1
   iroot
    10
   itda
    0
   istore # save TDDFT wave function in the 1st scratch file
    1
   $end
   
   #2nd: spin-flip tddft, use close-shell determinant as reference to calculate triplets 
   $tddft
   isf # notice here: ask for spin-flip up calculation
    1
   itda
    0
   idiag
    1
   iroot
    10
   istore # save TDDFT wave function in the 2nd scratch file, must be specified
    2
   $end
   
   #3rd: tddft-soc calculation
   $tddft
   isoc
    2
   nprt # print level
    10
   nfiles
    2
   ifgs # whether to include the ground state in the SOC treatment. 0=no, 1=yes
    1
   imatsoc
    8
    0 0 0 2 1 1
    0 0 0 2 2 1
    0 0 0 2 3 1
    0 0 0 2 4 1
    1 1 1 2 1 1
    1 1 1 2 2 1
    1 1 1 2 3 1
    1 1 1 2 4 1
   imatrso
    6
    1 1
    1 2
    1 3
    1 4
    1 5
    1 6
   idiag # full diagonalization of SO Hamiltonian
    2
   $end

.. warning:: 

  * The calculations must be performed in the order isf=0,isf=1. When SOC processing does not consider the ground state(i.e., ``ifgs=0``), the more excited states ``iroot``, the more accurate the result; when considering the ground state(i.e., ``ifgs=1``), too many ``iroot`` will reduce the accuracy, which is reflected in the underestimation of the ground state energy, and there is no fixed rule for the selection of ``iroot``.

The keyword ``imatsoc`` controls which SOC matrix elements <A|hso|B> are to be printed.

  * * ``8`` means that the SOCs between 8 sets of spin states are to be printed, and an array of 8 integer rows is entered sequentially below.
  * The input format for each line is ``fileA symA stateA fileB symB stateB``, representing the matrix element <fileA,symA,stateA|hsoc|fileB,symB,stateB>, where
  * ``fileA symA stateA`` represents the ``stateA`` root of the integrable representation of ``symA`` in ``fileA``; for example, ``1 1 1`` represents the first root of the integrable representation of the first TDDFT calculation.
  * ``0 0 0 0`` indicates the ground state

The printout of the coupling matrix element is as follows.

.. code-block:: 

    [tddft_soc_matsoc]

  Print selected matrix elements of [Hsoc] 

  SocPairNo. =    1   SOCmat = <  0  0  0 |Hso|  2  1  1 >     Dim =    1    3
    mi/mj          ReHso(au)       cm^-1               ImHso(au)       cm^-1
   0.0 -1.0      0.0000000000      0.0000000000      0.0000000000      0.0000000000
   0.0  0.0      0.0000000000      0.0000000000      0.0000000000      0.0000000000
   0.0  1.0      0.0000000000      0.0000000000      0.0000000000      0.0000000000

  SocPairNo. =    2   SOCmat = <  0  0  0 |Hso|  2  2  1 >     Dim =    1    3
    mi/mj          ReHso(au)       cm^-1               ImHso(au)       cm^-1
   0.0 -1.0      0.0000000000      0.0000000000      0.0000000000      0.0000000000
   0.0  0.0      0.0000000000      0.0000000000      0.0007155424    157.0434003237
   0.0  1.0      0.0000000000      0.0000000000     -0.0000000000     -0.0000000000

  SocPairNo. =    3   SOCmat = <  0  0  0 |Hso|  2  3  1 >     Dim =    1    3
    mi/mj          ReHso(au)       cm^-1               ImHso(au)       cm^-1
   0.0 -1.0     -0.0003065905    -67.2888361761      0.0000000000      0.0000000000
   0.0  0.0      0.0000000000      0.0000000000     -0.0000000000     -0.0000000000
   0.0  1.0     -0.0003065905    -67.2888361761     -0.0000000000     -0.0000000000

Here, ``< 0 0 0 |Hso| 2 2 1 >`` denotes the matrix element ``<S0|Hso|T1>`` , which gives its real part ReHso and imaginary part ImHso, respectively.
Since S0 has only one component, mi is 1. T1 (spin S=1) has 3 components (Ms=-1,0,1), and the 3 components are numbered by mj. 
The imaginary part of the coupling matrix element between the component with ``Ms=0`` and the ground state is ``0.0007155424 au`` .

.. warning::
  When comparing the results of different programs, note that the so-called spherical tensor is given here instead of cartesian tensor, i.e.,T1 is T_{-1},T_{0},T_{1}, not Tx,Ty,Tz， and there are you-transformations between them.  

The SOC calculation results in that

.. code-block:: 

        Totol No. of States:   161  Print:    10
  
    No.     1    w=     -0.0006 eV
         Spin: |Gs,1>    1-th Spatial:  A1;  OmegaSF=      0.0000eV  Cr=  0.0000  Ci=  0.9999  Per:100.0%
       SumPer: 100.0%
  
    No.     2    w=      1.5481 eV
         Spin: |S+,1>    1-th Spatial:  A2;  OmegaSF=      1.5485eV  Cr=  0.9998  Ci= -0.0000  Per:100.0%
       SumPer: 100.0%
  
    No.     3    w=      1.5482 eV
         Spin: |S+,3>    1-th Spatial:  A2;  OmegaSF=      1.5485eV  Cr=  0.9998  Ci=  0.0000  Per:100.0%
       SumPer: 100.0%
  
    No.     4    w=      1.5486 eV
         Spin: |S+,2>    1-th Spatial:  A2;  OmegaSF=      1.5485eV  Cr=  0.9999  Ci=  0.0000  Per:100.0%
       SumPer: 100.0%
  
    No.     5    w=      2.2106 eV
         Spin: |So,1>    1-th Spatial:  A2;  OmegaSF=      2.2117eV  Cr= -0.9985  Ci=  0.0000  Per: 99.7%
       SumPer:  99.7%
  
    No.     6    w=      2.5233 eV
         Spin: |S+,1>    1-th Spatial:  A1;  OmegaSF=      2.5232eV  Cr=  0.9998  Ci=  0.0000  Per:100.0%
       SumPer: 100.0%
  
    No.     7    w=      2.5234 eV
         Spin: |S+,3>    1-th Spatial:  A1;  OmegaSF=      2.5232eV  Cr=  0.9998  Ci= -0.0000  Per:100.0%
       SumPer: 100.0%
  
    No.     8    w=      2.5240 eV
         Spin: |S+,2>    1-th Spatial:  A1;  OmegaSF=      2.5232eV  Cr=  0.0000  Ci= -0.9985  Per: 99.7%
       SumPer:  99.7%
  
    No.     9    w=      5.5113 eV
         Spin: |S+,1>    1-th Spatial:  B2;  OmegaSF=      5.5115eV  Cr= -0.7070  Ci= -0.0000  Per: 50.0%
         Spin: |S+,3>    1-th Spatial:  B2;  OmegaSF=      5.5115eV  Cr=  0.7070  Ci=  0.0000  Per: 50.0%
       SumPer: 100.0%
  
    No.    10    w=      5.5116 eV
         Spin: |S+,1>    1-th Spatial:  B2;  OmegaSF=      5.5115eV  Cr= -0.5011  Ci= -0.0063  Per: 25.1%
         Spin: |S+,2>    1-th Spatial:  B2;  OmegaSF=      5.5115eV  Cr=  0.7055  Ci=  0.0000  Per: 49.8%
         Spin: |S+,3>    1-th Spatial:  B2;  OmegaSF=      5.5115eV  Cr= -0.5011  Ci= -0.0063  Per: 25.1%
       SumPer: 100.0%
  
   *** List of SOC-SI results ***
  
    No.      ExEnergies            Dominant Excitations         Esf        dE      Eex(eV)     (cm^-1) 
  
      1      -0.0006 eV   100.0%  Spin: |Gs,1>    0-th   A1    0.0000   -0.0006    0.0000         0.00
      2       1.5481 eV   100.0%  Spin: |S+,1>    1-th   A2    1.5485   -0.0004    1.5487     12491.27
      3       1.5482 eV   100.0%  Spin: |S+,3>    1-th   A2    1.5485   -0.0004    1.5487     12491.38
      4       1.5486 eV   100.0%  Spin: |S+,2>    1-th   A2    1.5485    0.0001    1.5492     12494.98
      5       2.2106 eV    99.7%  Spin: |So,1>    1-th   A2    2.2117   -0.0011    2.2112     17834.44
      6       2.5233 eV   100.0%  Spin: |S+,1>    1-th   A1    2.5232    0.0002    2.5239     20356.82
      7       2.5234 eV   100.0%  Spin: |S+,3>    1-th   A1    2.5232    0.0002    2.5239     20356.99
      8       2.5240 eV    99.7%  Spin: |S+,2>    1-th   A1    2.5232    0.0008    2.5246     20362.08
      9       5.5113 eV    50.0%  Spin: |S+,1>    1-th   B2    5.5115   -0.0002    5.5119     44456.48
     10       5.5116 eV    49.8%  Spin: |S+,2>    1-th   B2    5.5115    0.0001    5.5122     44458.63
     
Here the output has two parts, the first part gives the energy and composition of each ``SOC-SI`` state relative to the S0 state, for example

 * ``No. 10 w= 5.5116 eV`` means that the energy of the 10th ``SOC-SI`` state is ``5.5116 eV``, note that this is the energy relative to the S0 state;

The following three lines show the components of this state.

 * ``Spin: |S+,1> 1-th Spatial: B2;`` represents the first triplet state with symmetry B2 (spin +1 with respect to the S state, and therefore S+); and and therefore S+);
 *  ``OmegaSF= 5.5115eV`` is the energy relative to the first spin state.
 * ``Cr= -0.5011 Ci= -0.0063`` is the real and imaginary part of the wave function of this component in the spinor state, with a percentage of ``25.1%``.

The second part summarizes the results of the SOC-SI state calculations.

 * ``ExEnergies`` are the excitation energies when SOC is taken into account, and ``Esf`` is the original excitation energy without SOC;
 * The excited states are denoted by ``Spin: |S,M> n-th sym``, and spin |Gs,1>, the nth state with spatial symmetry sym. For example, the |Gs,1> represents the ground state, |So,1> represents the excited state with the same total spin as the ground state, and |S+,2> represents the excited state with total spin plus 1. M is the first component of the spin projection (in total 2S+1).
  
The keyword ``imatrso`` specifies which sets of jump dipole moments between the spin states are to be calculated and printed. Here it is specified that ``6`` sets of jump dipole moments are printed.
 * ``1 1`` indicates the intrinsic dipole moment of the ground state.
 * ``1 2`` indicates the dipole moments between the first and second spin states.

The output of the leap dipole moments is as follows.

.. code-block:: 

   [tddft_soc_matrso]: Print selected matrix elements of [dpl] 
  
    No.  ( I , J )   |rij|^2       E_J-E_I         fosc          rate(s^-1)
   -------------------------------------------------------------------------------
     1     1    1   0.472E+00    0.000000000    0.000000000     0.000E+00
     Details of transition dipole moment with SOC (in a.u.):
                     <I|X|J>       <I|Y|J>       <I|Z|J>        (also in debye) 
            Real=  -0.113E-15    -0.828E-18     0.687E+00    -0.0000  -0.0000   1.7471
            Imag=  -0.203E-35     0.948E-35     0.737E-35    -0.0000   0.0000   0.0000
            Norm=   0.113E-15     0.828E-18     0.687E+00
  
  
  
    No.  ( I , J )   |rij|^2       E_J-E_I         fosc          rate(s^-1)
   -------------------------------------------------------------------------------
     2     1    2   0.249E-05    1.548720567    0.000000095     0.985E+01
     Details of transition dipole moment with SOC (in a.u.):
                     <I|X|J>       <I|Y|J>       <I|Z|J>        (also in debye) 
            Real=  -0.589E-03     0.207E-07    -0.177E-15    -0.0015   0.0000  -0.0000
            Imag=  -0.835E-08     0.147E-02    -0.198E-16    -0.0000   0.0037  -0.0000
            Norm=   0.589E-03     0.147E-02     0.178E-15
  
  

.. hint::
  * ``imatsoc`` set to ``-1`` specifies printing of all coupling matrix elements;
  * The default is not to print the leap dipole moments, set ``imatrso`` to ``-1`` to print the leap dipole moments between all spin states, and ``imatrso`` to ``-2`` to print the leap dipole moments between all ground state spin states and all excited state spin states.
  * The reference state for SOC calculations must be either RHF/RKS or ROHF/ROKS, UHF/UKS is not supported.
  * When the reference state for SOC calculations is ROHF/ROKS, TDDFT calculations with isf=0 must use X-TDA (i.e., itest=1, icorrect=1,isf=0, itda=1; full X-TDDFT is not supported) and TDDFT calculations with isf=1 must use SF-TDA (i.e. isf=1, itda=1; full SF-TDDFT is not supported).


SOECP-TDDFT-SOC based spin-orbit coupling calculations
----------------------------------------------------------

In addition to the sf-X2C all-electron scalar relativistic Hamiltonian, the spin-orbit coupling pseudopotential (SOECP) can also be used for TDDFT-SOC spin-orbit coupling calculations by selecting the appropriate  :ref:`旋轨耦合赝势基组 <soecp-bas>` and setting ``hsoc`` to 0 in the ``xuanyuan`` module (if other values are written, they are treated as 0). 
The other inputs are similar to or identical to the sf-X2C-TDDFT-SOC inputs (e.g. the core electrons are subtracted when specifying the orbital occupation in ``scf``).

In the following example, the closed-shell layer ground state :math:`X^1\Sigma^+` (A1) and three excited states :math:`^3\Pi` (B1+B2),  :math:`^1\Pi` (B1+B2), :math:`^3\Sigma^+` (A1) are calculated under the :math:`C_{2v}` point group symmetry for the InB molecule, where the first two Λ-S states are bound states that have been extensively studied experimentally and the last two Λ-S states are repulsive states that are of little experimental interest.
In the input, the energy of the Λ-S state is first calculated at the TDDFT level (using the Tamm-Dancoff approximation here) and the wave function is stored, and then the energy of the Ω state after spin-orbit coupling is calculated.

.. code-block:: bdf

  $COMPASS
  Title
   soecp test: InBr
  Basis-block
    cc-pVTZ-PP
  end basis
  Geometry
    In  0.0  0.0  0.0
    Br  0.0  0.0  2.45
  END geometry
  group
   C(2v)      # Abelian symmetry must be used for SOC
  $END
  
  $XUANYUAN
   hsoc
    10
  $END
  
  $scf
    rks
    dft
     pbe0
  $end
  
  $TDDFT
  ISF
   0
  ITDA
   1
  istore
   1
  # 1Pi state: A1, A2, B1, B2
  nroot
    0 0 1 1
  $END
  
  $TDDFT
  ISF
   1
  ITDA
   1
  istore
   2
  # 3Sigma+ and 3Pi states: A1, A2, B1, B2
  nroot
    1 0 1 1
  $END
  
  $TDDFT
  isoc
   2
  nfiles
   2
  ifgs
   1
  idiag
   2
  $END

The computational output of SOECP-TDDFT-SOC is similar to that of sf-X2C-TDDFT-SOC. The results are summarized below and compared with those of EOM-CCSD/SOC.

.. table:: InBr分子的垂直激发能：SOECP/TDDFT-SOC与二分量EOM-CCSD。能量单位：cm :math:`^{-1}`
    :widths: auto
    :class: longtable

    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    |  Λ-S态              |    TDDFT    | Ω态 |   TDDFT-SOC |      分裂   |二分量EOM-CCSD|      分裂   |
    +=====================+=============+=====+=============+=============+==============+=============+
    | :math:`X^1\Sigma^+` |        0    | 0+  |         0   |             |         0    |             |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    | :math:`^3\Pi`       |    25731    | 0-  |     24884   |             |     24516    |             |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    |                     |             | 0+  |     24959   |        75   |     24588    |        72   |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    |                     |             | 1   |     25718   |       759   |     25363    |       775   |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    |                     |             | 2   |     26666   |       948   |     26347    |       984   |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    | :math:`^1\Pi`       |    35400    | 1   |     35404   |             |     36389    |             |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    | :math:`^3\Sigma^+`  |    38251    | 0-  |     38325   |             |              |             |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+
    |                     |             | 1   |     38423   |        98   |              |             |
    +---------------------+-------------+-----+-------------+-------------+--------------+-------------+

In addition to the SOECP basis set, the above calculation can also be done using the scalar ECP basis set in combination with :ref:`有效核电荷近似（Zeff）<so1e-zeff>` .
As a test, first delete the SO pseudopotential part in the Br basis set and redo the above calculation, but you will find that the result is poor:
The split of :math:`^3\Pi_2` and :math:`^3\Pi_1` is only 850 cm :math:`^{-1}`, while the split of :math:`^3\Sigma^+` almost zero.
This is because the ECP basis set for Br with 10 core electrons has no specially optimized effective nuclear charge, and the program can only take the actual nuclear charge number of 35:

.. code-block::

  SO-1e[BP] 
            Zeff for Wso
  ----------------------------------
   IAtm     ZA    NCore         Zeff
  ----------------------------------
      1     49       28        SOECP
      2     35       10         N.A.
  ----------------------------------

For Br in the above example, it is possible to use the scalar ECP basis set cc-pVTZ-ccECP with 28 core electrons instead. The input part of the basis set is modified as follows:

.. code-block:: bdf

   Basis-block
     cc-pvtz-pp
     Br=cc-pvtz-ccecp
   end basis

At the beginning of the TDDFT-SOC calculation output can be seen

.. code-block::

  SO-1e[BP] 
            Zeff for Wso
  ----------------------------------
   IAtm     ZA    NCore         Zeff
  ----------------------------------
      1     49       28        SOECP
      2     35       28     1435.000
  ----------------------------------

This shows that in the single-electron spin-orbit integration of Br, the default nuclear charge number 35 is replaced with the optimized 1435.000 (in general, the larger the ECP core electron number NCore, the larger the effective nuclear charge Zeff),
The SOECP integral is still calculated for the In atom. The calculation results are as follows, and it can be seen that the orbital splitting has been significantly improved:

.. table:: InBr分子的TDDFT-SOC垂直激发能：In:SOECP，Br:SOECP与Br:ECP。能量单位：cm :math:`^{-1}`
    :widths: auto
    :class: longtable

    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    |  Λ-S态              |    TDDFT    | Ω态 |   Br:SOECP  |      分裂   |     Br:ECP  |      分裂   |
    +=====================+=============+=====+=============+=============+=============+=============+
    | :math:`X^1\Sigma^+` |        0    | 0+  |         0   |             |         0   |             |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    | :math:`^3\Pi`       |    25731    | 0-  |     24884   |             |     25019   |             |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    |                     |             | 0+  |     24959   |        75   |     25084   |        65   |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    |                     |             | 1   |     25718   |       759   |     25856   |       772   |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    |                     |             | 2   |     26666   |       948   |     26808   |       952   |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    | :math:`^1\Pi`       |    35400    | 1   |     35404   |             |     35729   |             |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    | :math:`^3\Sigma^+`  |    38251    | 0-  |     38325   |             |     38788   |             |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+
    |                     |             | 1   |     38423   |        98   |     38853   |        65   |
    +---------------------+-------------+-----+-------------+-------------+-------------+-------------+

Finally, TDDFT-SOC calculations can also be combined with the SOECP (or scalar ECP) basis set with the all-electron non-relativistic basis set. The BDF program has optimized Zeff for main group elements prior to Xe (except heavier noble gas elements).
For example, continuing to use cc-pVTZ-PP for In, and using the all-electron non-relativistic basis set cc-pVTZ for Br, yields similar results to SOECP/TDDFT-SOC. Detailed results are omitted.Finally, TDDFT-SOC calculations can also be combined with the SOECP (or scalar ECP) basis set with the all-electron non-relativistic basis set. The BDF program has optimized Zeff for main group elements prior to Xe (except heavier noble gas elements).

.. attention::
  
   1. Precautions when using the effective nuclear charge method for TDDFT-SOC calculation: You must use :ref:`优化好的有效核电荷<so1e-zeff>` to ensure accuracy. To do this, check the Zeff value printed in the output file, try not to have N.A., this is especially important for ECP basis sets.
   2. When SOECP or scalar ECP is combined with all-electron basis set, note about all-electron basis set: Atoms using all-electron basis set do not consider scalar relativistic correspondence, so they cannot be heavy atoms, and must use non-relativistic basis set.

Calculation of first-order non-adiabatic coupling matrix elements (fo-NACME)
-------------------------------------------------------------------------------------------

As mentioned before, the (first-order) non-adiabatic coupling matrix element is of great importance in the non-radiative leap process. In BDF, the input files of NACME between the ground state and excited state, and between the excited state and excited state are written with some differences, which are described below.

(1) NACME between ground state and excited state: D0/D1 NACME of :math:`\ce{NO3}` radical (GB3LYP/cc-pVDZ)

.. code-block:: bdf

  $COMPASS
  Title
   NO3 radical NAC, 1st excited state
  Basis
   cc-pvdz
  Geometry
  N              0.0000000000         0.0000000000        -0.1945736441
  O             -2.0700698389         0.0000000000        -1.1615808530
  O              2.0700698389        -0.0000000000        -1.1615808530
  O             -0.0000000000         0.0000000000         2.4934136445
  End geometry
  unit
   bohr
  $END

  $XUANYUAN
  $END

  $SCF
  UKS
  dft
   GB3LYP
  spinmulti
   2
  $END

  $tddft
  iroot
   1 # One root for each irrep
  istore
   1 # File number, to be used later in $resp
  crit_vec
   1.d-6
  crit_e
   1.d-8
  gridtol
   1.d-7 # tighten the tolerance value of XC grid generation. This helps to
         # reduce numerical error, and is recommended for open-shell molecules
  $end

  $resp
  iprt
   1
  QUAD # quadratic response
  FNAC # first-order NACME
  single # calculation of properties from single residues (ground state-excited
         # state fo-NACMEs belong to this kind of properties)
  norder
   1
  method
   2
  nfiles
   1 # must be the same as the istore value in the $TDDFT block
  states
   1 # Number of excited states for which NAC is requested.
  # First number 1: read TDDFT results from file No. 1
  # Second number 2: the second irrep, in this case A2
  #   (note: this is the pair symmetry of the particle-hole pair, not
  #   the excited state symmetry. One must bear this in mind because the
  #   ground state of radicals frequently does not belong to the totally
  #   symmetric irrep)
  #   If no symmetry is used, simply use 1.
  # Third number 1: the 1st excited state that belongs to this irrep
   1 2 1
  $end

Note that the integrable representation specified in the ``$resp`` module is pair irrep (i.e., the direct product of the integrable representations of the occupied and empty orbitals involved in the leap; for the Abelian point group, pair irrep can be obtained from the direct product of the ground state integrable representation and the excited state integrable representation), not the irrep of the excited state. the ground state (D0) of this molecule belongs to the B1 integrable representation, and the first two-state excited state (D1) belongs to the B1 integrable representation. The pair irrep of the D1 state is therefore the direct product of B1 and B2, i.e., A2. Pair irrep can also be read from the output of the TDDFT module, i.e., the Pair column of the following output section.

.. code-block::

    No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

      1  A2    1  B2    0.8005 eV   1548.84 nm   0.0000   0.0186  98.2% CO(bb):  B2(   2 )->  B1(   5 )   3.992 0.622    0.0000
      2  B1    1  A1    1.9700 eV    629.35 nm   0.0011   0.0399  92.2% CO(bb):  A1(   8 )->  B1(   5 )   3.958 0.667    1.1695
      3  B2    1  A2    2.5146 eV    493.06 nm   0.0000   0.0384  98.4% CO(bb):  A2(   1 )->  B1(   5 )   4.159 0.319    1.7141
      4  A1    2  B1    2.6054 eV    475.87 nm   0.0171   0.0154  87.7% CO(bb):  B1(   4 )->  B1(   5 )   3.984 0.746    1.8049

After the calculation is completed, the result of the NACME calculation can be seen at the end of the output section of the ``$resp`` module.

.. code-block::

    Gradient contribution from Final-NAC(R)-Escaled
       1        0.0000000000       -0.0000000000        0.0000000000
       2       -0.0000000000       -0.1902838724        0.0000000000
       3       -0.0000000000        0.1902838724        0.0000000000
       4       -0.0000000000        0.0000000000        0.0000000000

Note that this result does not include the contribution of the electron translation factor (ETF). For some molecules, NACME without ETF may not have translation invariance, which may lead to errors in subsequent kinetic simulations. In this case, it is necessary to use a NACME that takes into account the ETF, which can be read later in the output file at the following location.

.. code-block::

    Gradient contribution from Final-NAC(S)-Escaled
       1        0.0000000000       -0.0000000000        0.0000000000
       2       -0.0000000000       -0.1920053581        0.0000000000
       3       -0.0000000000        0.1920053581        0.0000000000
       4       -0.0000000000        0.0000000000       -0.0000000000

The program will also output vectors named dpq-R, Final-NAC(R), dpq-S, Final-NAC(S), etc. These quantities are intermediate variables that are only used to monitor the computational process, not the final NACME, and the user can generally ignore these outputs.

(2) NACME between excited and excited states: T1/T2 NACME of acetophenone (BH&HLYP/def2-SVP)

.. code-block:: bdf

  $compass
  title
   PhCOMe
  basis
   def2-SVP
  geometry
          C             -0.3657620861         4.8928163606         0.0000770328
          C             -2.4915224786         3.3493223987        -0.0001063823
          C             -2.2618953860         0.7463412225        -0.0001958732
          C              0.1436118499        -0.3999193588        -0.0000964543
          C              2.2879147462         1.1871091769         0.0000824391
          C              2.0183382809         3.7824607425         0.0001740921
          H             -0.5627800515         6.9313968857         0.0001389666
          H             -4.3630645857         4.1868310874        -0.0002094148
          H             -3.9523568496        -0.4075513123        -0.0003833263
          H              4.1604797959         0.3598389310         0.0001836001
          H              3.6948496439         4.9629708946         0.0003304312
          C              0.3897478526        -3.0915327760        -0.0002927344
          O              2.5733215239        -4.1533492423        -0.0002053903
          C             -1.8017552120        -4.9131221777         0.0003595831
          H             -2.9771560760        -4.6352720097         1.6803279168
          H             -2.9780678476        -4.6353463569        -1.6789597597
          H             -1.1205416224        -6.8569277129         0.0002044899
  end geometry
  unit
   bohr
  nosymm
  $end

  $XUANYUAN
  $END

  $SCF
  rks
  dft
   bhhlyp
  $END

  $tddft
  isf # request for triplets (spin flip up)
   1
  ialda # use collinear kernel (NAC only supports collinear kernel)
   4
  iroot
   2 # calculate T1 and T2 states
  crit_vec
   1.d-6
  crit_e
   1.d-8
  istore
   1
  iprt
   2
  $end

  $resp
  iprt
   1
  QUAD
  FNAC
  double # calculation of properties from double residues (excited state-excited
         # state fo-NACMEs belong to this kind of properties)
  norder
   1
  method
   2
  nfiles
   1
  pairs
   1 # Number of pairs of excited states for which NAC is requested.
   1 1 1 1 1 2
  noresp # do not include the quadratic response contributions (recommended)
  $end

NACME was calculated for the T1 and T2 states.

.. code-block::

    Gradient contribution from Final-NAC(R)-Escaled
       1        0.0005655253        0.0005095355       -0.2407937116
       2       -0.0006501682       -0.0005568029        0.5339003311
       3        0.0009640605        0.0003767996       -2.6530192038
       4       -0.0013429266       -0.0034063171        1.6760344312
       5        0.0010446538        0.0006384285       -0.8024123329
       6       -0.0001081722       -0.0006245719       -0.0487310115
       7       -0.0000001499        0.0000176176       -0.0730900968
       8       -0.0000214634        0.0000165092        0.3841606239
       9        0.0000026057       -0.0000025322       -0.2553378323
      10       -0.0002028358       -0.0000591642        0.5800987974
      11       -0.0000166820        0.0000105734        0.2713836450
      12       -0.0023404123        0.0052038311        3.5121827769
      13        0.0021749503       -0.0012164868       -2.7480141157
      14        0.0000433873       -0.0011202812        0.2896243729
      15        0.1407516324        0.1432264573       -0.1655701318
      16       -0.1407399684       -0.1429881941       -0.1657943551
      17       -0.0000034197        0.0004577563       -0.0833951446

Similar to the case of the ground state, the

Definitization of the excited states
----------------------------------------------

.. code-block:: bdf

   $COMPASS
   Basis
    cc-pvdz
   Geometry
     C      0.000000    0.000000  0.000000
     C      1.332000    0.000000  0.000000
     H     -0.574301   -0.928785  0.000000
     H     -0.574301    0.928785  0.000000
     H      1.906301    0.928785  0.000000
     H      1.906301   -0.928785  0.000000
     C     -0.000000    0.000000  3.5000
     C      1.332000   -0.000000  3.5000
     H     -0.574301    0.928785  3.50000
     H     -0.574301   -0.928785  3.50000
     H      1.906301   -0.928785  3.50000
     H      1.906301    0.928785  3.50000
   End geometry
   Group
    C(1)
   Nfragment # must input: number of fragment, should be 1
    1
   $END
   
   $xuanyuan
   $end
   
   $scf
   rks
   dft 
    B3lyp
   $end
   
   $TDDFT
   ITDA
    1
   IDIAG
    1
   istore
    1
   iroot
     4
   crit_e # set a small threshhold for TDDFT energy convergence
     1.d-8
   $END
   
   # calculate local excited states (LOCALES) 
   $elecoup
   locales
     1
   $END
   
   &database
   fragment 1  12 # first fragment with 12 atoms, next line gives the atom list 
    1 2 3 4 5 6 7 8 9 10 11 12
   &end

The TDA calculates 4 excited states and the output is as follows,

.. code-block:: bdf

   No. Pair   ExSym   ExEnergies  Wavelengths      f     D<S^2>          Dominant Excitations             IPA   Ova     En-E1

    1   A    2   A    7.4870 eV    165.60 nm   0.0000   0.0000  82.6%  CV(0):   A(  16 )->   A(  17 )  13.476 0.820    0.0000
    2   A    3   A    8.6807 eV    142.83 nm   0.0673   0.0000  79.6%  CV(0):   A(  16 )->   A(  18 )  14.553 0.375    1.1937
    3   A    4   A    9.0292 eV    137.31 nm   0.0000   0.0000  62.4%  CV(0):   A(  16 )->   A(  20 )  15.353 0.398    1.5422
    4   A    5   A    9.0663 eV    136.75 nm   0.0000   0.0000  50.4%  CV(0):   A(  15 )->   A(  18 )  15.688 0.390    1.5793

The process of domainization and the excited states in the fixed domain are,

.. code-block:: bdf

      No.  8 iteration
    Pair States :    1   2
    aij,bij,abij -.25659893E-01 -.48045653E-11 0.25659893E-01
    cos4a,sin4a 0.10000000E+01 -.18724027E-09
    cosa,sina 0.10000000E+01 0.00000000E+00
    Pair States :    1   3
    aij,bij,abij -.40325646E-02 0.35638586E-11 0.40325646E-02
    cos4a,sin4a 0.10000000E+01 0.88376974E-09
    cosa,sina 0.10000000E+01 0.00000000E+00
    Pair States :    1   4
    aij,bij,abij -.25679319E-01 -.28753641E-08 0.25679319E-01
    cos4a,sin4a 0.10000000E+01 -.11197197E-06
    cosa,sina 0.10000000E+01 0.27877520E-07
    Pair States :    2   3
    aij,bij,abij -.39851115E-02 -.27118892E-05 0.39851124E-02
    cos4a,sin4a 0.99999977E+00 -.68050506E-03
    cosa,sina 0.99999999E+00 0.17012628E-03
    Pair States :    2   4
    aij,bij,abij -.42686102E-02 -.95914926E-06 0.42686103E-02
    cos4a,sin4a 0.99999997E+00 -.22469825E-03
    cosa,sina 0.10000000E+01 0.56174562E-04
    Pair States :    3   4
    aij,bij,abij -.67873307E-01 -.47952471E-02 0.68042488E-01
    cos4a,sin4a 0.99751360E+00 -.70474305E-01
    cosa,sina 0.99984454E+00 0.17632279E-01
    Sum=      0.13608498 Max Delta=      0.00531009
    
      No.  9 iteration
    Pair States :    1   2
    aij,bij,abij -.40325638E-02 0.35621782E-11 0.40325638E-02
    cos4a,sin4a 0.10000000E+01 0.88335323E-09
    cosa,sina 0.10000000E+01 0.00000000E+00
    Pair States :    1   3
    aij,bij,abij -.25690755E-01 -.11200070E-08 0.25690755E-01
    cos4a,sin4a 0.10000000E+01 -.43595721E-07
    cosa,sina 0.10000000E+01 0.10536712E-07
    Pair States :    1   4
    aij,bij,abij -.25690755E-01 -.10900573E-11 0.25690755E-01
    cos4a,sin4a 0.10000000E+01 -.42429944E-10
    cosa,sina 0.10000000E+01 0.00000000E+00
    Pair States :    2   3
    aij,bij,abij -.41480079E-02 -.83549288E-06 0.41480080E-02
    cos4a,sin4a 0.99999998E+00 -.20142027E-03
    cosa,sina 0.10000000E+01 0.50355067E-04
    Pair States :    2   4
    aij,bij,abij -.41480100E-02 0.17462423E-06 0.41480100E-02
    cos4a,sin4a 0.10000000E+01 0.42098314E-04
    cosa,sina 0.10000000E+01 0.10524580E-04
    Pair States :    3   4
    aij,bij,abij -.68042492E-01 0.19389042E-08 0.68042492E-01
    cos4a,sin4a 0.10000000E+01 0.28495490E-07
    cosa,sina 0.10000000E+01 0.74505806E-08
    Sum=      0.13608498 Max Delta=      0.00000000

    ***************** Diabatic Hamiltonian matrix ****************
                  State1      State2      State3     State4  
       State1    7.486977    0.000000    0.000000    0.000000
       State2    0.000000    9.029214   -0.000020    0.000021
       State3    0.000000   -0.000020    8.873501    0.192803
       State4    0.000000    0.000021    0.192803    8.873501
    **************************************************************

其中，对角元为定域激发态的能量，非对角元为两个定域态之间的耦合，这里的能量单位是 ``eV`` 。where the diagonal element is the energy of the fixed-domain excited state and the non-diagonal element is the coupling between the two fixed-domain states, where the unit of energy is ``eV``.
