Self-consistent field methods: Hartree-Fock and Kohn-Sham
===========================================

The self-consistent fields of the BDF include the Hartree-Fock and Kohn-Sham methods.

Restricted Hartree-Fock Method
-----------------------------------------------------------------

An example of the Restricted Hartree-Fock method (RHF) was mentioned in the :ref:`first example section <FirstExample>` and will not be repeated here.

Unrestricted Hartree-Fock method
-----------------------------------------------------------------

For systems with unpaired electrons, the ``UHF`` method is required, and the restricted open-shell Hartree-Fock (RHF) method can also be used, see later. 
For odd-electron systems, the BDF defaults to a spin multiplet of 2 and uses the UHF calculation. For example, to calculate the :math:`\ce{C3H5}` molecule，

.. code-block:: bdf

    #!bdf.sh
    UHF/3-21G 

    geometry
    C                  0.00000000    0.00000000    0.00000000
    C                  0.00000000    0.00000000    1.45400000
    C                  1.43191047    0.00000000    1.20151555
    H                  0.73667537   -0.61814403   -0.54629970
    H                 -0.90366611    0.32890757   -0.54629970
    H                  2.02151364    0.91459433    1.39930664
    H                  2.02151364   -0.91459433    1.39930664
    H                 -0.79835551    0.09653770    2.15071009
    end geometry

The output of the UHF calculation is similar to that of the RHF, in that the output from the ``scf`` module can be checked for correct charge and spin multiplicity.

.. code-block:: 

    Wave function information ...
    Total Nuclear charge    :      23
    Total electrons         :      23
    Ecp-core electrons      :       0
    Spin multiplicity(2S+1) :       2
    Num. of alpha electrons :      12
    Num. of beta  electrons :      11

The orbital occupancy is given separately for ``Alpha`` and ``Beta`` orbitals.

.. code-block:: 

    [Final occupation pattern: ]
    
     Irreps:        A   
    
     detailed occupation for iden/irep:      1   1
        1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00
        1.00 1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00
     Alpha      12.00
    
     detailed occupation for iden/irep:      2   1
        1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00 1.00
        1.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
        0.00 0.00 0.00 0.00 0.00 0.00 0.00
     Beta       11.00
    
The orbital energy, ``HOMO-LUMO gap`` is also printed separately according to ``Alpha`` and ``Beta`` orbits.

.. code-block:: 

    [Orbital energies:]
   
    Energy of occ-orbsA:    A            12
     -11.18817955    -11.18789391    -11.17752809     -1.11801069     -0.85914580
      -0.78861789     -0.65514687     -0.61300160     -0.55514631     -0.49906127
      -0.37655522     -0.30477047
    Energy of vir-orbsA:    A            25
       0.18221017      0.28830234      0.31069644      0.32818004      0.35397043
       0.38822931      0.42917813      0.49394022      0.93909970      0.94842069
       0.96877856      0.97277131      1.02563249      1.05399606      1.11320732
       1.17687697      1.26547430      1.31245896      1.32719078      1.34493766
       1.37905664      1.45903968      1.80285556      1.93877012      2.01720415
   
   
    Energy of occ-orbsB:    A            11
      -11.19670896   -11.16769083    -11.16660825     -1.07470168     -0.84162305
      -0.74622771     -0.63695581     -0.58068095     -0.53876236     -0.46400924
      -0.37745766
    Energy of vir-orbsB:    A            26
       0.15755278      0.18938428      0.30608423      0.33204779      0.33996597
       0.38195612      0.39002159      0.43644421      0.52237314      0.94876233
       0.96144960      0.97568581      1.01804430      1.05467405      1.09547593
       1.13390456      1.19865205      1.28139866      1.32654541      1.33938005
       1.34914150      1.38200544      1.47565481      1.79509704      1.96917149
       2.03513467
   
    Alpha   HOMO energy:      -0.30477047 au      -8.29322996 eV  Irrep: A       
    Alpha   LUMO energy:       0.18221017 au       4.95819299 eV  Irrep: A       
    Beta    HOMO energy:      -0.37745766 au     -10.27114977 eV  Irrep: A       
    Beta    LUMO energy:       0.15755278 au       4.28723115 eV  Irrep: A       
    HOMO-LUMO gap:       0.46232325 au      12.58046111 eV

Other output information can be found in the example of RHF calculation and will not be described in detail here.

Restricted open-shell Hartree-Fock method
------------------------------------------------------------------------------------------

Restricted open-shell Hartree-Fock（Restricted open-shell Hartree-Fock - ROHF）can also be calculated for the open-shell molecular system. An example of ROHF calculation for the :math:`\ce{CH2}` triplet state is given here.

.. code-block:: bdf

    #!bdf.sh
    rohf/cc-pvdz spinmulti=3
    
    geometry   # 输入坐标单位 Angstrom
     C     0.000000        0.00000        0.31399
     H     0.000000       -1.65723       -0.94197
     H     0.000000        1.65723       -0.94197
    end geometry

Here, the ``ROHF`` is specified in the second line and the triplet state is calculated using the keyword ``spinmulti=3``。The output of ROHF is similar to that of The output of ROHF is similar to UHF, but its ``Alpha`` orbitals are the same as ``Beta`` so the corresponding ``Alpha`` 和 ``Beta`` orbitals are equal in energy, as follows.

.. code-block:: 

    [Orbital energies:]
   
    Energy of occ-orbsA:    A1            3
      -11.42199273    -0.75328533     -0.22649749
    Energy of vir-orbsA:    A1            8
      0.05571960       0.61748052      0.70770696      0.83653819      1.29429307
      1.34522491       1.56472153      1.87720054
    Energy of vir-orbsA:    A2            2
      1.34320056       1.53663810
   
    Energy of occ-orbsA:    B1            1
     -0.37032603
    Energy of vir-orbsA:    B1            6
      0.06082087       0.66761691      0.77091474      1.23122892      1.51131609
      1.91351353
   
    Energy of occ-orbsA:    B2            1
     -0.16343739
    Energy of vir-orbsA:    B2            3
      0.65138659       1.35768658      1.54657952
   
   
    Energy of occ-orbsB:    A1            2
      -11.42199273    -0.75328533
    Energy of vir-orbsB:    A1            9
       -0.22649749     0.05571960      0.61748052      0.70770696      0.83653819
        1.29429307     1.34522491      1.56472153      1.87720054
    Energy of vir-orbsB:    A2            2
        1.34320056     1.53663810
   
    Energy of occ-orbsB:    B1            1
       -0.37032603
    Energy of vir-orbsB:    B1            6
        0.06082087     0.66761691      0.77091474      1.23122892      1.51131609
        1.91351353
    Energy of vir-orbsB:    B2            4
       -0.16343739     0.65138659      1.35768658      1.54657952
                 
Due to the different occupation numbers of ``Alpha`` and ``Beta`` orbitals, the HOMO, LUMO orbitals, and orbital energies of ``Alpha`` iffer from those of ``Beta``, as follows.
.. code-block:: 

    Alpha   HOMO energy:      -0.16343739 au      -4.44735961 eV  Irrep: B2      
    Alpha   LUMO energy:       0.05571960 au       1.51620803 eV  Irrep: A1      
    Beta    HOMO energy:      -0.37032603 au     -10.07708826 eV  Irrep: B1      
    Beta    LUMO energy:      -0.22649749 au      -6.16331290 eV  Irrep: A1      
    HOMO-LUMO gap:      -0.06306010 au      -1.71595329 eV


RKS, UKS, and ROKS Calculations
-------------------------------------------------
For the Restricted Kohn-Sham (RKS) method, an example of the RKS calculation for an :math:`\ce{H2O}` molecule is given here in a concise input mode, using the B3lyp generalized function.

.. code-block:: bdf

  #!bdf.sh
  B3lyp/3-21G    

  geometry
  O
  H  1  R1 
  H  1  R1  2 109.

  R1=1.0     # OH bond length, unit is Angstrom
  end geometry

The corresponding advanced mode input is: 

.. code-block:: bdf

    $compass
    geometry # On default: bond length unit in angstrom
    o
    h 1 1.0
    h 1 1.0 2 109.
    end geometry
    basis
      3-21g
    $end

    $xuanyuan
    $end

    $scf
    rks # Restricted Kohn-Sham calculation
    dft # ask for B3lyp functional, it is different with B3lyp implemented in Gaussian. 
      b3lyp
    $end

Here, the input requires the use of the ``B3lyp`` generic function. Compared to Hartree-Fock, the output has an additional contribution from the Exc term, as follows.

.. code-block:: 

   Final scf result
     E_tot =               -75.93603354
     E_ele =               -84.72787022
     E_nn  =                 8.79183668
     E_1e  =              -122.04354727
     E_ne  =              -197.45852687
     E_kin =                75.41497960
     E_ee  =                44.81744844
     E_xc  =                -7.50177140
    Virial Theorem      2.006909

The ROKS calculation for the :math:`\ce{H2O+}` ion is succinctly entered as follows.

.. code-block:: bdf

    #!bdf.sh
    ROKS/B3lyp/cc-pvdz charge=1    
    
    geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom 
    end geometry

.. hint::
    In contrast to Hartree-Fock，Kohn-Sham requires the use of the dft keyword in advanced input to specify the exchange-related generic function. For concise input, only the exchange-dependent generic function and basis group need to be specified. The system will choose to use RKS or UKS depending on the spin state, and must be entered explicitly if ROKS is to be used. 


Kohn-Sham calculations based on RS heterogeneous generalizations
-------------------------------------------------

CAM-B3LYP and other RS hybridization generalization functions, dividing the Coulomb interaction into long and short ranges, the

.. math::

    \frac{1}{r_{12}} = \frac{1-[\alpha + \beta \cdot erf(\mu r_{12})]}{r_{12}}+\frac{\alpha + \beta \cdot erf(\mu r_{12})}{r_{12}}

When using the BDF advanced input, the :math:`\mu` parameter can be adjusted using the keyword RS in the xuanyuan module. the default :math:`\mu` parameter for CAM-B3lyp is 0.33.
other :math:`\mu` values in RS Hybrid GGA see :ref:`RSOMEGA<xuanyuan_rsomega>` keyword。
for example, for 1,3-Butadiene molecules, the RKS advanced mode input using CAM-B3lyp is.

.. code-block:: bdf

   $compass
   basis
    cc-pVDZ
   geometry
   C -2.18046929 0.68443844 -0.00725330
   H -1.64640852 -0.24200621 -0.04439369
   H -3.24917614 0.68416040 0.04533562
   C -1.50331750 1.85817167 -0.02681816
   H -0.43461068 1.85844971 -0.07940766
   C -2.27196552 3.19155924 0.02664018
   H -3.34067218 3.19128116 0.07923299
   C -1.59481380 4.36529249 0.00707382
   H -2.12887455 5.29173712 0.04421474
   H -0.52610710 4.36557056 -0.04551805
   end geometry
   $end
   
   $xuanyuan
   rs
    0.33   # define mu=0.33 in CAM-B3lyp functional
   $end
   
   $scf
   rks # restricted Kohn-Sham
   dft
    cam-b3lyp
   $end


Exact exchange term and correlation term components for custom heterogeneous generalized functions, double heterogeneous generalized functions
-----------------------------------------------------------

For some calculations, it may be necessary for the user to manually adjust the exact exchange term components of the generic function to obtain satisfactory accuracy. In this case, you can add the ``facex`` keyword to the ``$scf`` module, for example, to change the exact exchange term component of the B3LYP generic function from the default 20% to 15%, you can write
.. code-block:: bdf

   $scf
   ...
   dft
    b3lyp
   facex
    0.15
   $end

Similarly, it is possible to customize the MP2-related term components of a two-hybrid generic function with the ``facco`` keyword. Note that not all generic functions support custom facex and facco（see :ref:`the SCF module for a list of keywords <scf>` ）。

Dispersion correction for weak interactions
-------------------------------------------------
Common exchange-correlation general functions such as B3lyp do not describe weak interactions well and require dispersion correction when calculating energy or doing molecular structure optimization. BDF uses the D3 dispersion correction method developed by Stefan Grimme, which requires specifying the D3 keyword in the input to the SCF module, as follows

.. code-block:: bdf

    #!bdf.sh
    B3lyp/cc-pvdz     
    
    geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom 
    end geometry
    
    $scf
    D3   # Grimme's dispersion correction
    $end

.. tip::

    * The BDF mixed-mode input method is used here to precisely control the SCF calculation by adding the SCF module keyword on top of the simple input.


The dispersion correction is added at the end of the Kohn-Sham calculation, and the calculated output is as follows.

.. code-block:: 

    diis/vshift is closed at iter =   8
    9    0   0.000  -76.380491166  -0.000000000  0.000000017  0.000000168  0.0000   0.02
   
     Label              CPU Time        SYS Time        Wall Time
    SCF iteration time:         0.467 S        0.033 S        0.233 S
   
    Final DeltaE =  -7.5459638537722640E-011
    Final DeltaD =   1.6950036756030376E-008   5.0000000000000002E-005
   
    Final scf result
      E_tot =               -76.38106481
      E_ele =               -85.17290149
      E_disp=                -0.00057364
      E_nn  =                 8.79183668
      E_1e  =              -122.51287853
      E_ne  =              -198.42779201
      E_kin =                75.91491348
      E_ee  =                44.84995532
      E_xc  =                -7.50940464
     Virial Theorem      2.006140

Here the total energy ``E_tot`` includes the dispersion correction energy, ``E_disp = -0.00057364`` 。


Improving the accuracy of integration lattice points for Kohn-Sham calculations
-------------------------------------------------

Although the BDF defines default integration grid points for different general functions according to their accuracy requirements (e.g., the Meta-GGA class of general functions requires high integration grid points, and the BDF defaults to Fine grid points), 
the user may also wish to adjust the integration grid points. The valid values of the Grid are ``Ultra coarse`` ，``Coarse`` ， ``medium`` ， ``fine`` ， ``Ultra fine`` ， ``sg1`` etc. From ``Ultra coarse`` to ``sg1``, the number of integration points increases and the numerical integration accuracy increases.

Example: M062X calculation of :math:`\ce{H2O}` molecule. This generalized function is a heterogeneous Meta-GGA type generalized function, which requires a dense integration grid, so the input uses a mixture of advanced input and simple input, as shown below.

.. code-block:: bdf

    #!bdf.sh
    M062X/cc-pvdz     
    
    geometry
    O
    H  1  R1
    H  1  R1  2 109.
    
    R1=1.0     # OH bond length in angstrom 
    end geometry
    
    $scf
    grid # set numerical integration grid as ultra fine
     ultra fine
    $end

BThe BDF uses ``Ultra coarse`` integration lattice points at the beginning steps of the Kohn-Sham calculation, as shown below.

.. code-block:: 

    Switch to Ultra Coarse grid ...
    [ATOM SCF control]
     heff=                     0
    After initial atom grid ...
    After initial atom grid ...
   
     Generating Numerical Integration Grid.
   
      1  O     Second Kind Chebyshev ( 21)  Lebedev ( -194)         
         Atoms:      1
      2  H     Second Kind Chebyshev ( 21)  Lebedev ( -194)         
         Atoms:      2     3
    Partition Function:  SSF   Partitioning with Scalar=  0.64.
    Gtol, Npblock, Icoulpot, Iop_adaptive :  0.10E-04    128      0          0
    Number of symmetry operation =   4
   
    Basis Informations for Self-adaptive Grid Generation, Cutoff=  0.10E-04
       1O     GTO( 14) Ntot=  26 MaxL= 2 MaxNL= 0 MaxRad= 0.530E+01
     basis details in form ( N L Zeta Cutradius): 
     ( 1  0   0.117E+05   0.02)  ( 1  0   0.176E+04   0.06)  ( 1  0   0.401E+03   0.13)
     ( 1  0   0.114E+03   0.24)  ( 1  0   0.370E+02   0.42)  ( 1  0   0.133E+02   0.70)
     ( 1  0   0.503E+01   1.14)  ( 1  0   0.101E+01   2.53)  ( 1  0   0.302E+00   4.64)
     ( 2  1   0.177E+02   0.66)  ( 2  1   0.385E+01   1.42)  ( 2  1   0.105E+01   2.72)
     ( 2  1   0.275E+00   5.30)  ( 3  2   0.119E+01   2.73)
       2H     GTO(  5) Ntot=   7 MaxL= 1 MaxNL= 0 MaxRad= 0.730E+01
     basis details in form ( N L Zeta Cutradius): 
     ( 1  0   0.130E+02   0.71)  ( 1  0   0.196E+01   1.82)  ( 1  0   0.445E+00   3.82)
     ( 1  0   0.122E+00   7.30)  ( 2  1   0.727E+00   3.26)
     Numerical Grid Generated SUCCESSFULLY! 
    Total and symmetry independent Grid Number:      4352      1181

When the energy converges to within 0.01 Hartree, it switches to the ``Ultra fine`` integration grid point and the output is shown below.

.. code-block:: 

     Switch to Ultra Fine grid ...
     [ATOM SCF control]
      heff=                     0
     After initial atom grid ...
     After initial atom grid ...
    
      Generating Numerical Integration Grid.
    
       1  O     Second Kind Chebyshev (100)  Lebedev (-1202)         
          Atoms:      1
       2  H     Second Kind Chebyshev (100)  Lebedev (-1202)         
          Atoms:      2     3
     Partition Function:  SSF   Partitioning with Scalar=  0.64.
     Gtol, Npblock, Icoulpot, Iop_adaptive :  0.10E-04    128      0          0
     Number of symmetry operation =   4
    
     Basis Informations for Self-adaptive Grid Generation, Cutoff=  0.10E-04
        1O     GTO( 14) Ntot=  26 MaxL= 2 MaxNL= 0 MaxRad= 0.530E+01
      basis details in form ( N L Zeta Cutradius): 
      ( 1  0   0.117E+05   0.02)  ( 1  0   0.176E+04   0.06)  ( 1  0   0.401E+03   0.13)
      ( 1  0   0.114E+03   0.24)  ( 1  0   0.370E+02   0.42)  ( 1  0   0.133E+02   0.70)
      ( 1  0   0.503E+01   1.14)  ( 1  0   0.101E+01   2.53)  ( 1  0   0.302E+00   4.64)
      ( 2  1   0.177E+02   0.66)  ( 2  1   0.385E+01   1.42)  ( 2  1   0.105E+01   2.72)
      ( 2  1   0.275E+00   5.30)  ( 3  2   0.119E+01   2.73)
        2H     GTO(  5) Ntot=   7 MaxL= 1 MaxNL= 0 MaxRad= 0.730E+01
      basis details in form ( N L Zeta Cutradius): 
      ( 1  0   0.130E+02   0.71)  ( 1  0   0.196E+01   1.82)  ( 1  0   0.445E+00   3.82)
      ( 1  0   0.122E+00   7.30)  ( 2  1   0.727E+00   3.26)
      Numerical Grid Generated SUCCESSFULLY! 
     Total and symmetry independent Grid Number:     94208     24827

Here, the integration lattice points of both H and O atoms are 100*1202, where 100 is the number of radial lattice points and 1202 is the number of angular lattice points.

