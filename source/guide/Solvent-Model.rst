Solvation models
================================================

Solvation models are used to calculate the interaction between solutes and solvents,
and are generally divided into two types: implicit solvent models (continuous medium models) and explicit solvent models. In BDF, for the continuous solvent
model, we use ddCOSMO (domain-decomposition COSMO solvation model), and for the display solvent model we use the QM/MM method in combination with the pDymamo2.0 program package for the calculation.

Calculation of solvation effects
------------------------------------------------
BDF currently supports the calculation of base-state solvation effects, including HF and DFT methods. The following are the input files for the solventization effect calculation of formaldehyde molecules file:

.. code-block:: bdf

  $COMPASS
  Title
    ch2o Molecule test run
  Basis
    6-31g
  Geometry
    C    0.00000000    0.00000000   -0.54200000
    O    0.00000000    0.00000000    0.67700000
    H    0.00000000    0.93500000   -1.08200000
    H    0.00000000    -0.9350000  -1.08200000
  END geometry
  nosym
  unit
   ang
  $END

  $xuanyuan
  $END

  $SCF
  rks
  dft
    b3lyp
  solvent   #Solvation calculation 
    water    #Specify the solvent
  grid
    medium
  $END

Where the solvent keyword is added to the ``SCF`` to indicate that a ``solvent`` effect calculation is to be performed, followed by a line where the solvent type, in this
case ``water`` , can be entered.
The list of solvent types supported in BDF is as follows:


.. table::

   ========================== ============================= ================================== =============================
    Water                      :math:`{\epsilon}` =78.3553   Butanone                           :math:`{\epsilon}` =18.246
    Acetonitrile               :math:`{\epsilon}` =35.688    ButanoNitrile                      :math:`{\epsilon}` =24.291
    Methanol                   :math:`{\epsilon}` =32.613    ButylAmine                         :math:`{\epsilon}` =4.6178
    Ethanol                    :math:`{\epsilon}` =24.852    ButylEthanoate                     :math:`{\epsilon}` =4.9941
    IsoQuinoline               :math:`{\epsilon}` =11.00     CarbonDiSulfide                    :math:`{\epsilon}` =2.6105
    Quinoline                  :math:`{\epsilon}` =9.16      Cis-1,2-DiMethylCycloHexane        :math:`{\epsilon}` =2.06
    Chloroform                 :math:`{\epsilon}` =4.7113    Cis-Decalin                        :math:`{\epsilon}` =2.2139
    DiethylEther               :math:`{\epsilon}` =4.24      CycloHexanone                      :math:`{\epsilon}` =15.619
    Dichloromethane            :math:`{\epsilon}` =8.93      CycloPentane                       :math:`{\epsilon}` =1.9608
    DiChloroEthane             :math:`{\epsilon}` =10.125    CycloPentanol                      :math:`{\epsilon}` =16.989
    CarbonTetraChloride        :math:`{\epsilon}` =2.2280    CycloPentanone                     :math:`{\epsilon}` =13.58
    Benzene                    :math:`{\epsilon}` =2.2706    Decalin-mixture                    :math:`{\epsilon}` =2.196
    Toluene                    :math:`{\epsilon}` =2.3741    DiBromomEthane                     :math:`{\epsilon}` =7.2273
    ChloroBenzene              :math:`{\epsilon}` =5.6968    DiButylEther                       :math:`{\epsilon}` =3.0473
    NitroMethane               :math:`{\epsilon}` =36.562    DiEthylAmine                       :math:`{\epsilon}` =3.5766
    Heptane                    :math:`{\epsilon}` =1.9113    DiEthylSulfide                     :math:`{\epsilon}` =5.723
    CycloHexane                :math:`{\epsilon}` =2.0165    DiIodoMethane                      :math:`{\epsilon}` =5.32
    Aniline                    :math:`{\epsilon}` =6.8882    DiIsoPropylEther                   :math:`{\epsilon}` =3.38
    Acetone                    :math:`{\epsilon}` =20.493    DiMethylDiSulfide                  :math:`{\epsilon}` =9.6
    TetraHydroFuran            :math:`{\epsilon}` =7.4257    DiPhenylEther                      :math:`{\epsilon}` =3.73
    DiMethylSulfoxide          :math:`{\epsilon}` =46.826    DiPropylAmine                      :math:`{\epsilon}` =2.9112
    Argon                      :math:`{\epsilon}` =1.430     e-1,2-DiChloroEthene               :math:`{\epsilon}` =2.14
    Krypton                    :math:`{\epsilon}` =1.519     e-2-Pentene                        :math:`{\epsilon}` =2.051
    Xenon                      :math:`{\epsilon}` =1.706     EthaneThiol                        :math:`{\epsilon}` =6.667
    n-Octanol                  :math:`{\epsilon}` =9.8629    EthylBenzene                       :math:`{\epsilon}` =2.4339
    1,1,1-TriChloroEthane      :math:`{\epsilon}` =7.0826    EthylEthanoate                     :math:`{\epsilon}` =5.9867
    1,1,2-TriChloroEthane      :math:`{\epsilon}` =7.1937    EthylMethanoate                    :math:`{\epsilon}` =8.3310
    1,2,4-TriMethylBenzene     :math:`{\epsilon}` =2.3653    EthylPhenylEther                   :math:`{\epsilon}` =4.1797
    1,2-DiBromoEthane          :math:`{\epsilon}` =4.9313    FluoroBenzene                      :math:`{\epsilon}` =5.42
    1,2-EthaneDiol             :math:`{\epsilon}` =40.245    Formamide                          :math:`{\epsilon}` =108.94
    1,4-Dioxane                :math:`{\epsilon}` =2.2099    FormicAcid                         :math:`{\epsilon}` =51.1
    1-Bromo-2-MethylPropane    :math:`{\epsilon}` =7.7792    HexanoicAcid                       :math:`{\epsilon}` =2.6
    1-BromoOctane              :math:`{\epsilon}` =5.0244    IodoBenzene                        :math:`{\epsilon}` =4.5470
    1-BromoPentane             :math:`{\epsilon}` =6.269     IodoEthane                         :math:`{\epsilon}` =7.6177
    1-BromoPropane             :math:`{\epsilon}` =8.0496    IodoMethane                        :math:`{\epsilon}` =6.8650
    1-Butanol                  :math:`{\epsilon}` =17.332    IsoPropylBenzene                   :math:`{\epsilon}` =2.3712
    1-ChloroHexane             :math:`{\epsilon}` =5.9491    m-Cresol                           :math:`{\epsilon}` =12.44
    1-ChloroPentane            :math:`{\epsilon}` =6.5022    Mesitylene                         :math:`{\epsilon}` =2.2650
    1-ChloroPropane            :math:`{\epsilon}` =8.3548    MethylBenzoate                     :math:`{\epsilon}` =6.7367
    1-Decanol                  :math:`{\epsilon}` =7.5305    MethylButanoate                    :math:`{\epsilon}` =5.5607
    1-FluoroOctane             :math:`{\epsilon}` =3.89      MethylCycloHexane                  :math:`{\epsilon}` =2.024
    1-Heptanol                 :math:`{\epsilon}` =11.321    MethylEthanoate                    :math:`{\epsilon}` =6.8615
    1-Hexanol                  :math:`{\epsilon}` =12.51     MethylMethanoate                   :math:`{\epsilon}` =8.8377
    1-Hexene                   :math:`{\epsilon}` =2.0717    MethylPropanoate                   :math:`{\epsilon}` =6.0777
    1-Hexyne                   :math:`{\epsilon}` =2.615     m-Xylene                           :math:`{\epsilon}` =2.3478
    1-IodoButane               :math:`{\epsilon}` =6.173     n-ButylBenzene                     :math:`{\epsilon}` =2.36
    1-IodoHexaDecane           :math:`{\epsilon}` =3.5338    n-Decane                           :math:`{\epsilon}` =1.9846
    1-IodoPentane              :math:`{\epsilon}` =5.6973    n-Dodecane                         :math:`{\epsilon}` =2.0060
    1-IodoPropane              :math:`{\epsilon}` =6.9626    n-Hexadecane                       :math:`{\epsilon}` =2.0402
    1-NitroPropane             :math:`{\epsilon}` =23.73     n-Hexane                           :math:`{\epsilon}` =1.8819
    1-Nonanol                  :math:`{\epsilon}` =8.5991    NitroBenzene                       :math:`{\epsilon}` =34.809
    1-Pentanol                 :math:`{\epsilon}` =15.13     NitroEthane                        :math:`{\epsilon}` =28.29
    1-Pentene                  :math:`{\epsilon}` =1.9905    n-MethylAniline                    :math:`{\epsilon}` =5.96
    1-Propanol                 :math:`{\epsilon}` =20.524    n-MethylFormamide-mixture          :math:`{\epsilon}` =181.56
    2,2,2-TriFluoroEthanol     :math:`{\epsilon}` =26.726    n,n-DiMethylAcetamide              :math:`{\epsilon}` =37.781
    2,2,4-TriMethylPentane     :math:`{\epsilon}` =1.9358    n,n-DiMethylFormamide              :math:`{\epsilon}` =37.219
    2,4-DiMethylPentane        :math:`{\epsilon}` =1.8939    n-Nonane                           :math:`{\epsilon}` =1.9605
    2,4-DiMethylPyridine       :math:`{\epsilon}` =9.4176    n-Octane                           :math:`{\epsilon}` =1.9406
    2,6-DiMethylPyridine       :math:`{\epsilon}` =7.1735    n-Pentadecane                      :math:`{\epsilon}` =2.0333
    2-BromoPropane             :math:`{\epsilon}` =9.3610    n-Pentane                          :math:`{\epsilon}` =1.8371
    2-Butanol                  :math:`{\epsilon}` =15.944    n-Undecane                         :math:`{\epsilon}` =1.9910
    2-ChloroButane             :math:`{\epsilon}` =8.3930    o-ChloroToluene                    :math:`{\epsilon}` =4.6331
    2-Heptanone                :math:`{\epsilon}` =11.658    o-Cresol                           :math:`{\epsilon}` =6.76
    2-Hexanone                 :math:`{\epsilon}` =14.136    o-DiChloroBenzene                  :math:`{\epsilon}` =9.9949
    2-MethoxyEthanol           :math:`{\epsilon}` =17.2      o-NitroToluene                     :math:`{\epsilon}` =25.669
    2-Methyl-1-Propanol        :math:`{\epsilon}` =16.777    o-Xylene                           :math:`{\epsilon}` =2.5454
    2-Methyl-2-Propanol        :math:`{\epsilon}` =12.47     Pentanal                           :math:`{\epsilon}` =10.0
    2-MethylPentane            :math:`{\epsilon}` =1.89      PentanoicAcid                      :math:`{\epsilon}` =2.6924
    2-MethylPyridine           :math:`{\epsilon}` =9.9533    PentylAmine                        :math:`{\epsilon}` =4.2010
    2-NitroPropane             :math:`{\epsilon}` =25.654    PentylEthanoate                    :math:`{\epsilon}` =4.7297
    2-Octanone                 :math:`{\epsilon}` =9.4678    PerFluoroBenzene                   :math:`{\epsilon}` =2.029
    2-Pentanone                :math:`{\epsilon}` =15.2      p-IsoPropylToluene                 :math:`{\epsilon}` =2.2322
    2-Propanol                 :math:`{\epsilon}` =19.264    Propanal                           :math:`{\epsilon}` =18.5
    2-Propen-1-ol              :math:`{\epsilon}` =19.011    PropanoicAcid                      :math:`{\epsilon}` =3.44
    3-MethylPyridine           :math:`{\epsilon}` =11.645    PropanoNitrile                     :math:`{\epsilon}` =29.324
    3-Pentanone                :math:`{\epsilon}` =16.78     PropylAmine                        :math:`{\epsilon}` =4.9912
    4-Heptanone                :math:`{\epsilon}` =12.257    PropylEthanoate                    :math:`{\epsilon}` =5.5205
    4-Methyl-2-Pentanone       :math:`{\epsilon}` =12.887    p-Xylene                           :math:`{\epsilon}` =2.2705
    4-MethylPyridine           :math:`{\epsilon}` =11.957    Pyridine                           :math:`{\epsilon}` =12.978
    5-Nonanone                 :math:`{\epsilon}` =10.6      sec-ButylBenzene                   :math:`{\epsilon}` =2.3446
    AceticAcid                 :math:`{\epsilon}` =6.2528    tert-ButylBenzene                  :math:`{\epsilon}` =2.3447
    AcetoPhenone               :math:`{\epsilon}` =17.44     TetraChloroEthene                  :math:`{\epsilon}` =2.268
    a-ChloroToluene            :math:`{\epsilon}` =6.7175    TetraHydroThiophene-s,s-dioxide    :math:`{\epsilon}` =43.962
    Anisole                    :math:`{\epsilon}` =4.2247    Tetralin                           :math:`{\epsilon}` =2.771
    Benzaldehyde               :math:`{\epsilon}` =18.220    Thiophene                          :math:`{\epsilon}` =2.7270
    BenzoNitrile               :math:`{\epsilon}` =25.592    Thiophenol                         :math:`{\epsilon}` =4.2728
    BenzylAlcohol              :math:`{\epsilon}` =12.457    trans-Decalin                      :math:`{\epsilon}` =2.1781
    BromoBenzene               :math:`{\epsilon}` =5.3954    TriButylPhosphate                  :math:`{\epsilon}` =8.1781
    BromoEthane                :math:`{\epsilon}` =9.01      TriChloroEthene                    :math:`{\epsilon}` =3.422
    Bromoform                  :math:`{\epsilon}` =4.2488    TriEthylAmine                      :math:`{\epsilon}` =2.3832
    Butanal                    :math:`{\epsilon}` =13.45     Xylene-mixture                     :math:`{\epsilon}` =2.3879
    ButanoicAcid               :math:`{\epsilon}` =2.9931    z-1,2-DiChloroEthene               :math:`{\epsilon}` =9.2
   ========================== ============================= ================================== =============================

Entering the dielectric constant
--------------------------------------------------------

For solvents that are not in the table, you can enter the dielectric constant. The format is as follows:

.. code-block:: bdf 

  solvent
    user   #User-specified
  dielectric
    78.3553   #Enter the dielectric constant


.. note::

   Solvation effects are currently only supported for energy calculations, and gradient calculations will be completed in the near future. 


Excited State Solvation Effect
----------------------------------------------------------

The excited state solvation effect can be calculated using a combination of explicit and implicit solvents. Take an aqueous solution as an example, due to
the solute molecule's HOMO and LUMO orbitals may diffuse into the first hydration layer, so the excited state calculation can include the water molecules in the
first hydration layer in the TDDFT calculation region, while the rest is treated with an implicit solvent.

Take sinapic acid as an example. To determine the first hydration layer of the solute molecule, the Amber program can be used to simulate the molecular dynamics
of the mustard acid molecule in a small box of water. After the system is equilibrated, the distribution of water molecules around the solute molecule can
be analyzed to determine the first hydration layer. Of course, it is also possible to select a multi-frame structure for calculation and then take an average.

The selection of molecules in the hydration layer can be done using the VMD program. Assuming that the input is a pdb file, the first hydration layer molecule
can be selected on the command line and saved as a pdb file. The command is as follows:

.. code-block:: bdf 

  atomselect top  "same resid as (within 3.5  of not water)"   # Select the first hydration layer
  atomselect0 writepdb sa.pdb                     #Solute molecules and first hydrated layers are preserved in pdb files

In the above example, all water molecules within 3.5 ?? of the solute molecule are selected, and the entire molecule is selected as long as one of the three atoms
of the water molecule is within the truncation range. The selection results are shown in the figure:

.. figure:: /images/SAtddft.jpg

Based on the coordinate information in the sa.pdb file, the TDDFT calculation is performed with the following input file???

.. code-block:: bdf

  $COMPASS 
  Title
   SA Molecule test run
  Basis
   6-31g
  Geometry
  C          14.983  14.539   6.274
  C          14.515  14.183   7.629
  C          13.251  14.233   8.118
  C          12.774  13.868   9.480
  C          11.429  14.087   9.838
  C          10.961  13.725  11.118
  O           9.666  13.973  11.525
  C           8.553  14.050  10.621
  C          11.836  13.125  12.041
  O          11.364  12.722  13.262
  C          13.184  12.919  11.700
  O          14.021  12.342  12.636
  C          15.284  11.744  12.293
  C          13.648  13.297  10.427
  O          14.270  14.853   5.341
  O          16.307  14.468   6.130
  H          15.310  13.847   8.286
  H          12.474  14.613   7.454
  H          10.754  14.550   9.127
  H           7.627  14.202  11.188
  H           8.673  14.888   9.924
  H           8.457  13.118  10.054
  H          10.366  12.712  13.206
  H          15.725  11.272  13.177
  H          15.144  10.973  11.525
  H          15.985  12.500  11.922
  H          14.687  13.129  10.174
  H          16.438  14.756   5.181
  O          18.736   9.803  12.472
  H          18.779  10.597  11.888
  H          19.417  10.074  13.139
  O          18.022  14.021   8.274
  H          17.547  14.250   7.452
  H          18.614  13.310   7.941
  O           8.888  16.439   7.042
  H           9.682  16.973   6.797
  H           8.217  17.162   7.048
  O           4.019  14.176  11.140
  H           4.032  13.572  10.360
  H           4.752  14.783  10.885
  O          16.970   8.986  14.331
  H          17.578   9.273  13.606
  H          17.497   8.225  14.676
  O           8.133  17.541  10.454
  H           8.419  17.716  11.386
  H           8.936  17.880   9.990
  O           8.639  12.198  13.660
  H           7.777  11.857  13.323
  H           8.413  13.155  13.731
  O          13.766  11.972   4.742
  H          13.858  12.934   4.618
  H          13.712  11.679   3.799
  O          10.264  16.103  14.305
  H           9.444  15.605  14.054
  H          10.527  15.554  15.084
  O          13.269  16.802   3.701
  H          13.513  16.077   4.325
  H          14.141  17.264   3.657
  O          13.286  14.138  14.908
  H          13.185  14.974  14.393
  H          13.003  13.492  14.228
  O          16.694  11.449  15.608
  H          15.780  11.262  15.969
  H          16.838  10.579  15.161
  O           7.858  14.828  14.050
  H           7.208  15.473  13.691
  H           7.322  14.462  14.795
  O          15.961  17.544   3.706
  H          16.342  16.631   3.627
  H          16.502  17.866   4.462
  O          10.940  14.245  16.302
  H          10.828  13.277  16.477
  H          11.870  14.226  15.967
  O          12.686  10.250  14.079
  H          11.731  10.151  14.318
  H          12.629  11.070  13.541
  O           9.429  11.239   8.483
  H           8.927  10.817   7.750
  H           9.237  12.182   8.295
  O          17.151  15.141   3.699
  H          17.124  14.305   3.168
  H          18.133  15.245   3.766
  O          17.065  10.633   9.634
  H          16.918  10.557   8.674
  H          17.024   9.698   9.909
  O          17.536  14.457  10.874
  H          18.014  13.627  11.089
  H          17.683  14.460   9.890
  O           5.836  16.609  13.299
  H           4.877  16.500  13.549
  H           5.760  16.376  12.342
  O          19.014  12.008  10.822
  H          18.249  11.634  10.308
  H          19.749  11.655  10.256
  O          15.861  14.137  15.750
  H          14.900  13.990  15.574
  H          16.185  13.214  15.645
  O          11.084   9.639  10.009
  H          11.641   9.480   9.213
  H          10.452  10.296   9.627
  O          14.234  10.787  16.235
  H          13.668  10.623  15.444
  H          13.663  10.376  16.925
  O          14.488   8.506  13.105
  H          13.870   9.136  13.550
  H          15.301   8.683  13.628
  O          14.899  17.658   9.746
  H          15.674  18.005   9.236
  H          15.210  16.754   9.926
  O           8.725  13.791   7.422
  H           9.237  13.488   6.631
  H           8.845  14.770   7.309
  O          10.084  10.156  14.803
  H           9.498  10.821  14.366
  H          10.215  10.613  15.669
  O           5.806  16.161  10.582
  H           5.389  16.831   9.993
  H           6.747  16.470  10.509
  O           6.028  13.931   7.206
  H           5.971  14.900   7.257
  H           6.999  13.804   7.336
  O          17.072  12.787   2.438
  H          16.281  12.594   1.885
  H          17.062  11.978   3.013
  END geometry
  nosym
  mpec+cosx
  $END
  
  $xuanyuan
  $end
  
  $SCF
  rks
  dft
   b3lyp   
  solvent
   water 
  grid
   medium
  $END
  
  # input for tddft
  $tddft
  iroot    # Calculate 1 root for each irrep. By default, 10 roots are calculated
    1      # for each irrep
  memjkop  # maxium memeory for Coulomb and Exchange operator. 1024 MW (Mega Words)
    1024 
  $end





