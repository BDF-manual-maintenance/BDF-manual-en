Exchange-dependent generalized functions supported by BDF
===============================================
The density flooding theory (DFT) of the BDF supports restricted, unrestricted and restricted open-shell Kohn-Sham calculations, referred to as RKS, UKS and ROKS, respectively. The inputs are close to those of RHF, UHF and ROHF. The key is to specify the exchange-related generic functions. bdf supports a variety of generic functions such as LDA, GGA, Meta-GGA, Hybrid, RS Hybrid and Hyrid Meta-GGA.

.. table:: Functionals supported by BDF
    :widths: 40 60

    ========================================  ====================================================
     Functional type                                  Functionals
    ========================================  ====================================================
     Local-density approximation,LDA（LDA）    LSDA, SVWN5, SAOP
     Generalized gradient approximation（GGA） BP86, BLYP, PBE, PW91, OLYP, KT2
     meta-GGA                                  TPSS, M06L, M11L, MN12L, MN15L, SCAN, r2SCAN
     Hybrid GGA                                B3LYP, GB3LYP, BHHLYP, PBE0, B3PW91, HFLYP, VBLYP
     Range-separated Hybrid GGA                wB97, wB97X, CAM-B3LYP, LC-BLYP
     Hybrid Meta-GGA                           TPSSh, M062X, PW6B95
     Double Hybrid                             B2PLYP
    ========================================  ====================================================

.. attention::
    1. VWN5 is used for the LDA correlation term of B3LYP, while VWN3 is used for the LDA correlation term of GB3LYP corresponding to B3LYP in the Gaussian program 2。
    2. For range-separated generalized calculations, the ``rs`` values must be set manually in the ``Xuanyuan`` module（see :ref:`the list of keywords in the xuanyuan module <xuanyuan>` ）。wB97, wB97X, CAM-B3LYP and LC-BLYP have rs values of 0.40, 0.30, 0.33 and 0.33, respectively。
    3. For the two-hybrid generalized function calculation, an ``MP2`` module must be added after the ``SCF`` module（see example test116 in the :doc:`example description <Example>` ）and the final result is read from the output of the ``MP2`` module.
    4. User-defined generalized functions can be implemented in the ``SCF`` module by adjusting the proportion of HF exchange terms and the proportion of MP2 related terms in the generalized function using the ``facex`` and ``facco`` keywords（see :doc:`the list of keywords in the SCF module <scf>` ）。
    5. BDF uses libxc，in principle supports all the general functions included in libxc, but it takes time to refine and add. Users can give us feedback on the required generic functions so that we can add them as needed.
    
Note that while all general functions support (without dispersion correction) single-point energy calculations for the ground state, some functions are only partially supported by the general function are supported. The following is a list of the general functions supported for various computational tasks.


.. table:: BDF functions supported by different computing task types
    :widths: 30 70

    ======================== ===================================================================================================
     Calculation task type             functional
    ======================== ===================================================================================================
     TDDFT single point energy and SOC calculation     All functionals except double hybrid functionals
     TDDFT excited state dipole moment        LSDA, SVWN5, BP86, BLYP, PBE, OLYP, B3LYP, GB3LYP, BHHLYP, PBE0, HFLYP, CAM-B3LYP, LC-BLYP
     Ground state gradient                 All LDA、GGA、hybrid GGA functional、meta-GGA and hybrid meta GGA functional except SAOP、PW91、KT2、B3PW91、VBLYP、SF5050
     Excited state gradient、NAC         all LDA、GGA and hybrid GGA functional except SAOP、PW91、KT2、B3PW91、VBLYP、SF5050
     Energy transfer / electron transfer integral    All functionals are supported, but the results of B2PLYP do not include the contribution of MP2 related terms, so they are approximate
     NMR                           All LDA, GGA and hybrid GGA functionals
     Dispersion correction                 BP86, BLYP, PBE, B3LYP, GB3LYP, BHHLYP, B3PW91, PBE0, CAM-B3LYP, B2PLYP
    ======================== ===================================================================================================
    

    
