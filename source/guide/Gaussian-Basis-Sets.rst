Gaussian basis group
================================================

In order to solve the Hartree-Fock, Kohn-Sham DFT equations, it is necessary to expand the molecular orbitals into linear combinations of single-electron basis functions.

.. math::
    \varphi_{i}(r) = C_{1,i}\chi_{1}(r) + C_{2,i}\chi_{2}(r) + C_{3,i}\chi_{3}(r) + \dots + C_{N,i}\chi_{N}(r)

In quantum chemistry calculations, the basis functions have only mathematical meaning, not physical meaning. The more basis functions there are, the more accurate the result will be, but it also depends on how well the basis functions are set up. The Complete Basis Set Limit (CBS) is reached when there are infinitely many basis functions, which is called a complete set, and the molecular orbitals can be perfectly expanded. The actual use of a finite basis set does not reach the CBS, and the resulting error in the calculation result is called the basis set incompleteness error.

As many basis functions as are used, as many molecular orbitals are produced, but only occupied orbitals, and lower order non-occupied orbitals (valence level empty orbitals) are usually chemically meaningful. If the basis functions are taken to be atomic orbitals, it is called linear combination of atomic orbitals (LCAO), but this is only a concept in structural chemistry, and the basis functions used in actual calculations are not the real atomic orbitals.

The commonly used basis functions in quantum chemistry are as follows.

#. Gauss type orbital（GTO）basis functions: Most quantum chemistry programs use GTO basis functions because they are mathematically easy to calculate two-electron integrals.
#. Slater orbital (Slater type orbital, STO) basis functions: Semi-empirical and used by a few quantum chemistry programs (e.g. ADF). It is difficult to calculate two-electron integrals, but its radial behavior is closer to the actual atomic orbitals than the GTO basis functions, so that only a small number of STOs are needed to achieve a large number of GTO results.
#. Plane wave: A basis function specifically suitable for periodic calculations and much less cost effective than the GTO basis function for isolated systems.
#. Numerical atomic orbital (NAO) basis functions: Few programs support them, typically Dmol3, Siesta. NAO basis functions do not have an analytic mathematical form, but are described by discrete distributions of points.

The STO basis function was used in the early days of BDF software, and the GTO basis function is mainly used now.

For GTO basis functions with orbital angular momentum *L* higher than *p*（e.g., GTO basis functions such as *d* 、*f*, etc.），there are two ways to represent them.
One is written in the form of a Cartesian function (also called a right-angle function).

.. math::
   N x^{lx} y^{ly} z^{lz} {\rm exp}(-\alpha r^2),  \qquad L=lx+ly+lz

It has :math:`(L+1)(L+2)/2` components, e.g., the *d* function contains xx，yy，zz，xy，xz，yz。The other is written in the form of a spherical function (also called a spherical harmonic function, pure function).

.. math::
   N Y^L_m r^L {\rm exp}(-\alpha r^2)

It has :math:`2L+1` components, for example, the *d* function contains -2，-1，0，+1，+2。

The advantage of the Cartesian function is that it is easy to calculate the integral, but there are redundant functions; whereas the spherical function corresponds to exactly :math:`(L+1)(L+2)/2` magnetic quantum numbers,
so in quantum chemistry programs the integral is usually calculated first under the Cartesian function and then combined into the integral of the spherical function by a certain linear relation :cite:`schlegel1995`.

.. attention::

  1. most modern Gaussian basis groups are optimized under spherical basis functions, except for the older basis groups such as Pople type.
  2. Cartesian basis functions have no advantage in terms of accuracy or efficiency, especially for all-electron relativity calculations, which also lead to numerical instability, so spherical basis functions are always used in BDF calculations. 
  3. Cartesian and spherical basis functions lead to different results. If the results of BDF calculations are repeated with other quantum chemistry programs, it is necessary to check whether the spherical basis functions are used, in addition to ensuring that the structure, method, and basis group are the same. 
In the literature, data sets of optimized GTO basis functions for various atoms in different situations have been created and given different names to be called by quantum chemistry programs. These named GTO basis function data sets are called **Gaussian Basis Sets**。
the Gaussian Basis Sets built into the BDF are mainly from the following Basis Set Repository websites, and the original literature on the various Basis Sets can be found at the corresponding websites.

* Basis Set Exchange :cite:`bse2019` ：All-electron basisets, scalar ECP basisets, can be exported in BDF format（note: ECP basisets have to be manually repositioned for ECP data）。 https://www.basissetexchange.org/
* Stuttgart/Cologne pseudopotential basis set library: mainly SOECP basis sets, and a few early scalar ECP basis sets. http://www.tc.uni-koeln.de/PP/clickpse.en.html
* Turbomole basis set library:all-electron basis set, scalar ECP basis set, SOECP basis set. http://www.cosmologic-services.de/basis-sets/basissets.php
* Dyall Relativistic Basis Group: All-electron relativistic basis group. http://dirac.chem.sdu.dk/basisarchives/dyall/index.html
* Sapporo basis set library: all-electron basis sets. http://sapporo.center.ims.ac.jp/sapporo/
* Clarkson University ECP basis group library: SOECP basis group. https://people.clarkson.edu/~pchristi/reps.html
* ccECP Basis Group Library: Scalar ECP Basis Groups. https://pseudopotentiallibrary.org/

In addition, there are individual elements with built-in motifs from the original literature.

* All-electron basis set Dirac-RPF-4Z and Dirac-aug-RPF-4Z, including s-、p-region elements :cite:`dasilva2014`，d-region elements :cite:`dasilva2014a`，f-region elements :cite:`dasilva2017`
* Pseudopotential basis group Pitzer-AVDZ-PP、Pitzer-VDZ-PP、Pitzer-VTZ-PP :cite:`pitzer2000`
* Ce - Lu :cite:`ermler1994`, Fr - Pu :cite:`ermler1991`, Am - Og :cite:`ermler1997,ermler1999` in the pseudopotential basis group CRENBL（Note: the Am - Og basis group on the Basis Set Exchange is wrong!）
* Am - Og :cite:`ermler1997,ermler1999` in the pseudopotential basis group CRENBS（Note: the Am - Og basis set on Basis Set Exchange is wrong!）
* Ac, Th, Pa :cite:`dolg2014` ，U :cite:`dolg2009` in the pseudopotential basis group Stuttgart-ECPMDFSO-QZVP

BDF users can use either the standard basis sets from the BDF basis set library or custom basis sets.


.. _all-e-bas:

All-electron basis groups
------------------------------------------------

All-electron basis groups are divided into two categories: non-shrinking basis groups and shrinking basis groups. The former can be used for both non-relativistic and relativistic calculations, but mainly for relativistic calculations, while the latter is divided into non-relativistic shrinkage basis groups and relativistic shrinkage basis groups.

All-electron relativistic calculations use Hamiltonians such as DKH, ZORA, X2C, etc. that take relativistic effects into account（see :ref:`Relativistic effects <relativity>` ），
when it is necessary to use shrinkage basis groups optimized specifically for relativistic calculations, such as the cc-pVnZ-DK series, SARC, ANO-RCC, etc. 
Most relativistic shrinkage basis sets treat the nucleus as a point charge, but some do take into account the nucleus distribution size effect when doing the shrinkage, which has the most pronounced effect on the shrinkage factor of the *s* and *p* asis functions.
Accordingly, a finite nucleus model must also be used in the calculation of molecular integrals. :ref:`finite nucleus model <finite-nuclear>` 。

.. table:: all electron basis set in BDF basis set library
    :widths: auto
    :class: longtable

    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | basis set type         | basis set name              | supported element                      | note                   |
    +========================+=============================+========================================+========================+
    | Pople                  | | STO-3G                    | 1- 54                                  |                        |
    |                        | | STO-6G                    |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 3-21G                     | 1- 55                                  |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 3-21++G                   | 1,  3- 20                              |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-31G                     | 1- 36                                  |                        |
    |                        | | 6-31G(d,p)                |                                        |                        |
    |                        | | 6-31GP                    |                                        |                        |
    |                        | | 6-31GPP                   |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-31++G                   | 1- 20                                  |                        |
    |                        | | 6-31++GP                  |                                        |                        |
    |                        | | 6-31++GPP                 |                                        |                        |
    |                        | | 6-31+G                    |                                        |                        |
    |                        | | 6-31+GP                   |                                        |                        |
    |                        | | 6-31+GPP                  |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-31G(2df,p)              | 1- 18                                  |                        |
    |                        | | 6-31G(3df,3pd)            |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-311++G                  | 1,  3- 20                              |                        |
    |                        | | 6-311++G(2d,2p)           |                                        |                        |
    |                        | | 6-311++GP                 |                                        |                        |
    |                        | | 6-311++GPP                |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-311+G                   | 1- 20                                  |                        |
    |                        | | 6-311+G(2d,p)             |                                        |                        |
    |                        | | 6-311+GP                  |                                        |                        |
    |                        | | 6-311+GPP                 |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-311G                    | 1- 20, 31- 36, 53                      |                        |
    |                        | | 6-311G(d,p)               |                                        |                        |
    |                        | | 6-311GP                   |                                        |                        |
    |                        | | 6-311GPP                  |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-31++GPP-J               | 1,  6-  8                              |                        |
    |                        | | 6-31+GP-J                 |                                        |                        |
    |                        | | 6-31G-J                   |                                        |                        |
    |                        | | 6-311++GPP-J              |                                        |                        |
    |                        | | 6-311+GP-J                |                                        |                        |
    |                        | | 6-311G-J                  |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-311G(2df,2pd)           | 1- 10, 19- 20                          |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | 6-311++G(3df,3pd)         | 1,  3- 18                              |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | correlate consistency  | | aug-cc-pVDZ               | 1- 18, 21- 36                          |                        |
    |                        | | aug-cc-pVTZ               |                                        |                        |
    |                        | | aug-cc-pVQZ               |                                        |                        |
    |                        | | aug-cc-pV5Z               |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ                   | 1- 18, 20- 36                          |                        |
    |                        | | cc-pVTZ                   |                                        |                        |
    |                        | | cc-pVQZ                   |                                        |                        |
    |                        | | cc-pV5Z                   |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pV6Z               | 1-  2,  5- 10, 13- 18                  |                        |
    |                        | | cc-pV6Z                   |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pV7Z               | 5- 10                                  |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pCVDZ              | 1- 18                                  |                        |
    |                        | | aug-cc-pCVTZ              |                                        |                        |
    |                        | | aug-cc-pCVQZ              |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pCV5Z              | 5- 18                                  |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pCVDZ                  | 1- 18, 20                              |                        |
    |                        | | cc-pCVTZ                  |                                        |                        |
    |                        | | cc-pCVQZ                  |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pV(D+d)Z           | 1- 18, 21- 36                          |                        |
    |                        | | aug-cc-pV(T+d)Z           |                                        |                        |
    |                        | | aug-cc-pV(Q+d)Z           |                                        |                        |
    |                        | | aug-cc-pV(5+d)Z           |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pV(D+d)Z               | 1- 18, 20- 36                          |                        |
    |                        | | cc-pV(T+d)Z               |                                        |                        |
    |                        | | cc-pV(Q+d)Z               |                                        |                        |
    |                        | | cc-pV(5+d)Z               |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pwCVDZ             | | D: 5- 10, 13- 18                     |                        |
    |                        | | aug-cc-pwCVTZ             | | T: 5- 10, 13- 18, 21- 30             |                        |
    |                        | | aug-cc-pwCVQZ             | | Q: 5- 10, 13- 18, 21- 30, 35         |                        |
    |                        | | aug-cc-pwCV5Z             | | 5: 5- 10, 13- 18, 21- 30             |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pVDZ-RIFIT         | 1-  2,  4- 10, 12- 18, 21- 36          | auxiliary basis set    |
    |                        | | aug-cc-pVTZ-RIFIT         |                                        |                        |
    |                        | | aug-cc-pVQZ-RIFIT         |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pV5Z-RIFIT         | | 5: 1- 10, 13- 18, 21- 36             | auxiliary basis set    |
    |                        | | aug-cc-pV6Z-RIFIT         | | 6: 1-  2,  5- 10, 13- 18             |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pVTZ-J             | 1,  5-  9, 13- 17, 21- 30, 34          | auxiliary basis set    |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pVDZ-DK            | | D: 1- 18, 21- 36                     | relativistic effect    |
    |                        | | aug-cc-pVTZ-DK            | | T: 1- 18, 21- 36, 39- 46             |                        |
    |                        | | aug-cc-pVQZ-DK            | | Q: 1- 18, 21- 36                     |                        |
    |                        | | aug-cc-pV5Z-DK            | | 5: 1-  2,  5- 10, 13- 18, 21- 36     |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pCVDZ-DK           | 3- 18                                  | relativistic effect    |
    |                        | | aug-cc-pCVTZ-DK           |                                        |                        |
    |                        | | aug-cc-pCVQZ-DK           |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pwCVTZ-DK          | | T: 21- 30, 39- 46                    | relativistic effect    |
    |                        | | aug-cc-pwCVQZ-DK          | | Q: 21- 30                            |                        |
    |                        | | aug-cc-pwCV5Z-DK          | | 5: 21- 30                            |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pVDZ-DK3           | | D: 55- 56, 87- 88                    |  relativistic effect    |
    |                        | | aug-cc-pVTZ-DK3           | | T: 49- 56, 81- 88                    |                        |
    |                        | | aug-cc-pVQZ-DK3           | | Q: 49- 56, 81- 88                    |                        |
    |                        | | aug-cc-pwCVDZ-DK3         |                                        |                        |
    |                        | | aug-cc-pwCVTZ-DK3         |                                        |                        |
    |                        | | aug-cc-pwCVQZ-DK3         |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pVDZ-X2C           | 19- 20, 37- 38, 55- 56, 87- 88         | relativistic effect    |
    |                        | | aug-cc-pVTZ-X2C           |                                        |                        |
    |                        | | aug-cc-pVQZ-X2C           |                                        |                        |
    |                        | | aug-cc-pwCVDZ-X2C         |                                        |                        |
    |                        | | aug-cc-pwCVTZ-X2C         |                                        |                        |
    |                        | | aug-cc-pwCVQZ-X2C         |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-DK                | | D: 1- 18, 21- 36                     |  relativistic effect   |
    |                        | | cc-pVTZ-DK                | | T: 1- 18, 21- 36, 39- 46             |                        |
    |                        | | cc-pVQZ-DK                | | Q: 1- 18, 21- 36                     |                        |
    |                        | | cc-pV5Z-DK                | | 5: 1- 18, 21- 36                     |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pwCVTZ-DK              | | T: 21- 30, 39- 46                    |  relativistic effect   |
    |                        | | cc-pwCVQZ-DK              | | Q: 21- 30                            |                        |
    |                        | | cc-pwCV5Z-DK              | | 5: 21- 30                            |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-DK3               | | D: 55- 71, 87-103                    |  relativistic effect   |
    |                        | | cc-pVTZ-DK3               | | T: 49- 71, 81-103                    |                        |
    |                        | | cc-pVQZ-DK3               | | Q: 49- 71, 81-103                    |                        |
    |                        | | cc-pwCVDZ-DK3             |                                        |                        |
    |                        | | cc-pwCVTZ-DK3             |                                        |                        |
    |                        | | cc-pwCVQZ-DK3             |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-X2C               | 19- 20, 37- 38, 55- 71, 87-103         | relativistic effect    |
    |                        | | cc-pVTZ-X2C               |                                        |                        |
    |                        | | cc-pVQZ-X2C               |                                        |                        |
    |                        | | cc-pwCVDZ-X2C             |                                        |                        |
    |                        | | cc-pwCVTZ-X2C             |                                        |                        |
    |                        | | cc-pwCVQZ-X2C             |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-FW_fi             | 1-2,  5-10, 13-18, 31-36               |  relativistic effect,  |
    |                        | | cc-pVTZ-FW_fi             |                                        |  finite nucleus model  |
    |                        | | cc-pVQZ-FW_fi             |                                        |                        |
    |                        | | cc-pV5Z-FW_fi             |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-FW_pt             | 1-2,  5-10, 13-18, 31-36               | relativistic effect    |
    |                        | | cc-pVTZ-FW_pt             |                                        |                        |
    |                        | | cc-pVQZ-FW_pt             |                                        |                        |
    |                        | | cc-pV5Z-FW_pt             |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | ANO                    | | ADZP-ANO                  | 1-103                                  |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | ANO-DK3                   | 1- 10                                  |  relativistic effect   |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | ANO-R                     | 1- 86                                  |  relativistic effect,  |
    |                        | | ANO-R0                    |                                        | finite nucleus model   |
    |                        | | ANO-R1                    |                                        |                        |
    |                        | | ANO-R2                    |                                        |                        |
    |                        | | ANO-R3                    |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | ANO-RCC                   | 1- 96                                  | relativistic effect    |
    |                        | | ANO-RCC-VDZ               |                                        |                        |
    |                        | | ANO-RCC-VDZP              |                                        |                        |
    |                        | | ANO-RCC-VTZP              |                                        |                        |
    |                        | | ANO-RCC-VQZP              |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | ANO-RCC-VTZ               | 3- 20, 31- 38                          | relativistic effect     |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | Ahlrichs               | | Def2系列                  | 全电子非相对论基组与赝势基组的混合，see :ref:`pseudopotential basis set <ecp-bas>` |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | jorge-DZP                 | | D: 1-103                             |                        |
    |                        | | jorge-TZP                 | | T: 1-103                             |                        |
    |                        | | jorge-QZP                 | | Q: 1- 54                             |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | jorge-DZP-DKH             | | D: 1-103                             |relativistic effect    |
    |                        | | jorge-TZP-DKH             | | T: 1-103                             |                        |
    |                        | | jorge-QZP-DKH             | | Q: 1- 54                             |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | SARC-DKH2                 | 57- 86, 89-103                         | relativistic effect     |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | SARC2-QZV-DKH2            | 57- 71                                 | relativistic effect    |
    |                        | | SARC2-QZVP-DKH2           |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | x2c-SV(P)all              | 1- 86                                  | relativistic effect    |
    |                        | | x2c-SVPall                |                                        |                        |
    |                        | | x2c-TZVPall               |                                        |                        |
    |                        | | x2c-TZVPPall              |                                        |                        |
    |                        | | x2c-QZVPall               |                                        |                        |
    |                        | | x2c-QZVPPall              |                                        |                        |
    |                        | | x2c-SV(P)all-2c           |                                        |                        |
    |                        | | x2c-SVPall-2c             |                                        |                        |
    |                        | | x2c-TZVPall-2c            |                                        |                        |
    |                        | | x2c-TZVPPall-2c           |                                        |                        |
    |                        | | x2c-QZVPall-2c            |                                        |                        |
    |                        | | x2c-QZVPPall-2c           |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | Sapporo                | | Sapporo-DZP               | 1- 54                                  | 2012 newest version    |
    |                        | | Sapporo-TZP               |                                        |                        |
    |                        | | Sapporo-QZP               |                                        |                        |
    |                        | | Sapporo-DZP-2012          |                                        |                        |
    |                        | | Sapporo-TZP-2012          |                                        |                        |
    |                        | | Sapporo-QZP-2012          |                                        |                        |
    |                        | | Sapporo-DZP-dif           |                                        |                        |
    |                        | | Sapporo-TZP-dif           |                                        |                        |
    |                        | | Sapporo-QZP-dif           |                                        |                        |
    |                        | | Sapporo-DZP-2012-dif      |                                        |                        |
    |                        | | Sapporo-TZP-2012-dif      |                                        |                        |
    |                        | | Sapporo-QZP-2012-dif      |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Sapporo-DKH3-DZP          | 1- 54                                  | relativistic effect     |
    |                        | | Sapporo-DKH3-TZP          |                                        |                        |
    |                        | | Sapporo-DKH3-QZP          |                                        |                        |
    |                        | | Sapporo-DKH3-DZP-dif      |                                        |                        |
    |                        | | Sapporo-DKH3-TZP-dif      |                                        |                        |
    |                        | | Sapporo-DKH3-QZP-dif      |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Sapporo-DKH3-DZP-2012     | 19- 86                                 | relativistic effect    |
    |                        | | Sapporo-DKH3-TZP-2012     |                                        | finite nucleus model   |
    |                        | | Sapporo-DKH3-QZP-2012     |                                        |                        |
    |                        | | Sapporo-DKH3-DZP-2012-dif |                                        |                        |
    |                        | | Sapporo-DKH3-TZP-2012-dif |                                        |                        |
    |                        | | Sapporo-DKH3-QZP-2012-dif |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | non-contracted         | | UGBS                      | 1- 90, 94- 95, 98-103                  | relativistic effect     |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Dirac-RPF-4Z              | 1-118                                  | relativistic effect    |
    |                        | | Dirac-aug-RPF-4Z          |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Dirac-Dyall.2zp           | 1-118                                  |relativistic effect    |
    |                        | | Dirac-Dyall.3zp           |                                        |                        |
    |                        | | Dirac-Dyall.4zp           |                                        |                        |
    |                        | | Dirac-Dyall.ae2z          |                                        |                        |
    |                        | | Dirac-Dyall.ae3z          |                                        |                        |
    |                        | | Dirac-Dyall.ae4z          |                                        |                        |
    |                        | | Dirac-Dyall.cv2z          |                                        |                        |
    |                        | | Dirac-Dyall.cv3z          |                                        |                        |
    |                        | | Dirac-Dyall.cv4z          |                                        |                        |
    |                        | | Dirac-Dyall.v2z           |                                        |                        |
    |                        | | Dirac-Dyall.v3z           |                                        |                        |
    |                        | | Dirac-Dyall.v4z           |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Dirac-Dyall.aae2z         | | 1-2, 5-10, 13-18, 31-36, 49-54       | relativistic effect     |
    |                        | | Dirac-Dyall.aae3z         | | 81-86, 113-118                       |                        |
    |                        | | Dirac-Dyall.aae4z         |                                        |                        |
    |                        | | Dirac-Dyall.acv2z         |                                        |                        |
    |                        | | Dirac-Dyall.acv3z         |                                        |                        |
    |                        | | Dirac-Dyall.acv4z         |                                        |                        |
    |                        | | Dirac-Dyall.av2z          |                                        |                        |
    |                        | | Dirac-Dyall.av3z          |                                        |                        |
    |                        | | Dirac-Dyall.av4z          |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | others                 | | SVP-BSEX                  | 1, 3-10                                |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | DZP                       | 1, 6-8, 16, 26, 42                     |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | DZVP                      | 1, 3-9, 11-17, 19-20, 31-35, 49-53     |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | TZVPP                     | 1, 6-7                                 |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | IGLO-II                   | 1,  5-  9, 13- 17                      |                        |
    |                        | | IGLO-III                  |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Sadlej-pVTZ               | 1,  6- 8                               |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Wachters+f                | 21- 29                                 |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+


.. _ecp-bas:

Pseudopotential basis groups
------------------------------------------------

The Effective Core Potential (ECP) includes the Pseudopotential (PP) and the Model Core Potential (MCP).
The PP in quantum chemical calculations is not fundamentally different from the PP in plane wave calculations, except that it is expressed in a concise analytic form.
Most quantum chemistry software, including BDF, supports PP, but fewer quantum chemistry software support MCP, so the names ECP and PP can be used interchangeably without ambiguity.

The pseudopotential basis group needs to be used in conjunction with the pseudopotential, and the basis functions describe only the valence level electrons of the atoms. When heavier elements are involved in the system, the pseudopotential basis group is usually used for them, while the normal basis group is used for the other atoms as usual. The Lan series, the Stuttgart series, and the cc-pVnZ-PP series all belong to this group. For ease of recall, the pseudopotential basis groups of some lighter elements are actually non-relativistic all-electron basis groups, such as the Def2 series of basis groups for elements before the fifth period.

.. _soecp-bas:

The pseudopotential basis groups are divided into scalar pseudopotential basis groups and spin-orbit coupled pseudopotential (SOECP) basis groups, depending on whether the pseudopotential contains a spin-orbit coupling term or not.

.. table:: BDF基组库中的标准赝势基组
    :widths: auto
    :class: longtable

    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | 基组类型               | 基组名称                    | 支持的元素                             | 备注                   |
    +========================+=============================+========================================+========================+
    | 关联一致               | | aug-cc-pVDZ-PP            | 29- 36, 39- 54, 72- 86                 | SOECP                  |
    |                        | | aug-cc-pVTZ-PP            |                                        |                        |
    |                        | | aug-cc-pVQZ-PP            |                                        |                        |
    |                        | | aug-cc-pV5Z-PP            |                                        |                        |
    |                        | | aug-cc-pwCVDZ-PP          |                                        |                        |
    |                        | | aug-cc-pwCVTZ-PP          |                                        |                        |
    |                        | | aug-cc-pwCVQZ-PP          |                                        |                        |
    |                        | | aug-cc-pwCV5Z-PP          |                                        |                        |
    |                        | | cc-pV5Z-PP                |                                        |                        |
    |                        | | cc-pwCV5Z-PP              |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-PP                | 29- 36, 39- 54, 72- 86, 90- 92         | SOECP                  |
    |                        | | cc-pVTZ-PP                |                                        |                        |
    |                        | | cc-pVQZ-PP                |                                        |                        |
    |                        | | cc-pwCVDZ-PP              |                                        |                        |
    |                        | | cc-pwCVTZ-PP              |                                        |                        |
    |                        | | cc-pwCVQZ-PP              |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pCVDZ-ccECP        | 19- 30                                 |                        |
    |                        | | aug-cc-pCVTZ-ccECP        |                                        |                        |
    |                        | | aug-cc-pCVQZ-ccECP        |                                        |                        |
    |                        | | aug-cc-pCV5Z-ccECP        |                                        |                        |
    |                        | | cc-pCVDZ-ccECP            |                                        |                        |
    |                        | | cc-pCVTZ-ccECP            |                                        |                        |
    |                        | | cc-pCVQZ-ccECP            |                                        |                        |
    |                        | | cc-pCV5Z-ccECP            |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | aug-cc-pVDZ-ccECP         | | D: 3- 9, 11- 17, 19- 36              |                        |
    |                        | | aug-cc-pVTZ-ccECP         | | T: 3- 9, 11- 17, 19- 36              |                        |
    |                        | | aug-cc-pVQZ-ccECP         | | Q: 3- 9, 11- 17, 19- 36              |                        |
    |                        | | aug-cc-pV5Z-ccECP         | | 5: 3- 9, 11- 17, 19- 36              |                        |
    |                        | | aug-cc-pV6Z-ccECP         | | 6: 4- 9, 12- 17, 19- 20, 31- 36      |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | cc-pVDZ-ccECP             | | D: 3- 36                             |                        |
    |                        | | cc-pVTZ-ccECP             | | T: 3- 36                             |                        |
    |                        | | cc-pVQZ-ccECP             | | Q: 3- 36                             |                        |
    |                        | | cc-pV5Z-ccECP             | | 5: 3- 36                             |                        |
    |                        | | cc-pV6Z-ccECP             | | 6: 4- 10, 12- 20, 31- 36             |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Pitzer-AVDZ-PP            | 3- 10                                  | SOECP                  |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Pitzer-VDZ-PP             | 3- 18                                  | SOECP                  |
    |                        | | Pitzer-VTZ-PP             |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | Clarkson               | | CRENBL                    | 1 (全电子), 3-118                      | SOECP，小芯            |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | CRENBS                    | | 21- 36, 39- 54, 57, 72- 86,          | SOECP，大芯            |
    |                        |                             | | 104-118                              |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | Ahlrichs               | | Def2-SVP                  | 1- 36 (全电子), 37- 57, 72- 86         | TM73是新版             |
    |                        | | Def2-SV(P)                |                                        |                        |
    |                        | | Def2-SVPD                 |                                        |                        |
    |                        | | Def2-SVPD-TM73            |                                        |                        |
    |                        | | Def2-TZVP                 |                                        |                        |
    |                        | | Def2-TZVPD                |                                        |                        |
    |                        | | Def2-TZVPD-TM73           |                                        |                        |
    |                        | | Def2-TZVP-F               |                                        |                        |
    |                        | | Def2-TZVPP-F              |                                        |                        |
    |                        | | Def2-TZVPP                |                                        |                        |
    |                        | | Def2-TZVPPD               |                                        |                        |
    |                        | | Def2-TZVPPD-TM73          |                                        |                        |
    |                        | | Def2-QZVP                 |                                        |                        |
    |                        | | Def2-QZVPD                |                                        |                        |
    |                        | | Def2-QZVPD-TM73           |                                        |                        |
    |                        | | Def2-QZVPP                |                                        |                        |
    |                        | | Def2-QZVPPD               |                                        |                        |
    |                        | | Def2-QZVPPD-TM73          |                                        |                        |
    |                        | | ma-Def2-SV(P)             |                                        |                        |
    |                        | | ma-Def2-SVP               |                                        |                        |
    |                        | | ma-Def2-TZVP              |                                        |                        |
    |                        | | ma-Def2-TZVPP             |                                        |                        |
    |                        | | ma-Def2-QZVP              |                                        |                        |
    |                        | | ma-Def2-QZVPP             |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Def2-SV(P)-TM73           | 1- 36 (全电子), 37- 86                 | TM73是新版             |
    |                        | | Def2-SVP-TM73             |                                        |                        |
    |                        | | Def2-TZVP-TM73            |                                        |                        |
    |                        | | Def2-TZVPP-TM73           |                                        |                        |
    |                        | | Def2-TZVP-F-TM73          |                                        |                        |
    |                        | | Def2-TZVPP-F-TM73         |                                        |                        |
    |                        | | Def2-QZVP-TM73            |                                        |                        |
    |                        | | Def2-QZVPP-TM73           |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | DHF-SV(P)                 | 37- 56, 72- 86                         | SOECP                  |
    |                        | | DHF-SVP                   |                                        |                        |
    |                        | | DHF-TZVP                  |                                        |                        |
    |                        | | DHF-TZVPP                 |                                        |                        |
    |                        | | DHF-QZVP                  |                                        |                        |
    |                        | | DHF-QZVPP                 |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | LAN                    | | LANL2DZ                   | | 1, 3-10 (全电子)                     |                        |
    |                        |                             | | 11-57, 72-83, 92-94                  |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | LANL2DZDP                 | | 1, 6-9 (全电子)                      |                        |
    |                        |                             | | 14-17, 32-35, 50-53, 82-83           |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | LANL2TZ                   | 21- 30, 39- 48, 57, 72- 80             |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | LANL08                    | 11- 57, 72- 83                         |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | LANL08(D)                 | 14- 17, 32- 35, 50- 53, 82- 83         |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | LANL2TZ+                  | 21- 30                                 |                        |
    |                        | | LANL08+                   |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Modified-LANL2DZ          | 21- 29, 39- 47, 57, 72- 79             |                        |
    |                        | | LANL2TZ(F)                |                                        |                        |
    |                        | | LANL08(F)                 |                                        |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | SBKJC                  | | SBKJC-VDZ                 | 1-2 (全电子), 3- 58, 72- 86            |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | SBKJC-POLAR               | | 1-2 (全电子)                         |                        |
    |                        |                             | | 3- 20, 32- 38, 50- 56, 82- 86        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | pSBKJC                    | 6- 9, 14- 17, 32- 35, 50- 53           |                        |
    +------------------------+-----------------------------+----------------------------------------+------------------------+
    | Stuttgart              | | Stuttgart-RLC             | | 3- 20, 30- 38, 49- 56, 80- 86        |                        |
    |                        |                             | | 89-103                               |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Stuttgart-RSC-1997        | | 19-30, 37-48, 55-56, 58-70           |                        |
    |                        |                             | | 72-80, 89-103, 105                   |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Stuttgart-RSC-ANO         | 57- 71, 89-103                         | SOECP                  |
    |                        | | Stuttgart-RSC-SEG         |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Stuttgart-ECP92MDFQ-DZVP  | 111-120                                | SOECP                  |
    |                        | | Stuttgart-ECP92MDFQ-TZVP  |                                        |                        |
    |                        | | Stuttgart-ECP92MDFQ-QZVP  |                                        |                        |
    +                        +-----------------------------+----------------------------------------+------------------------+
    |                        | | Stuttgart-ECPMDFSO-QZVP   | 19- 20, 37- 38, 55- 56, 87- 92         | SOECP                  |
    +------------------------+-----------------------------+----------------------------------------+------------------------+

.. _SelfdefinedBasis:

自定义基组文件
------------------------------------------------
BDF可以使用非内置基组，此时要把基组数据保存在文本格式的基组文件中，放在计算目录下，文件名就是BDF中要引用的基组名。

.. warning::

    自定义基组文件的文件名必须 **全部大写** ！但在输入文件中引用时，大小写任意。

例如，在计算目录下创建一个文本文件MYBAS-1（注意：如果在Windows操作系统下创建文本文件，系统可能会隐去扩展名 *.txt* ，因此实际名称是MYBAS-1.txt），内容为：

.. code-block::

   # This is my basis set No. 1.               # 任意的空行，以及 # 打头的注释行 
   # Supported elements: He and Al

   ****                                        # 4个星号打头的行，接下来是一个元素的基组
   He      2    1                              # 元素符号，核电荷数，基函数的最高角动量
   S      4    2                               # S型GTO基函数，4个原函数收缩成2个
                  3.836000E+01                 # 4个S型高斯原函数的指数
                  5.770000E+00
                  1.240000E+00
                  2.976000E-01
         2.380900E-02           0.000000E+00   # 两列收缩因子，对应两个收缩的S型GTO基函数
         1.548910E-01           0.000000E+00
         4.699870E-01           0.000000E+00
         5.130270E-01           1.000000E+00
   P      2    2                               # P型GTO基函数，2个原函数收缩成2个
                  1.275000E+00
                  4.000000E-01
         1.0000000E+00           0.000000E+00
         0.0000000E+00           1.000000E+00
   ****                       # 4个星号结束He的基组，后面可接另一个元素的基组，或者结束
   Al     13    2
   （略）

在以上的基组中，P函数未作收缩，也可以写成以下形式：

.. code-block::

   （S函数，略）
   P      2    0              # 0表示非收缩，此时不需要提供收缩因子
                  1.275000E+00
                  4.000000E-01
   ****
   （略）

对于赝势基组，还需要在价基函数后提供ECP数据。例如，

.. code-block::

   ****                                              # 价基函数部分，注释同上
   Al     13    2
   S       4    3
              14.68000000
               0.86780000
               0.19280000
               0.06716000
       -0.0022368000     0.0000000000     0.0000000000
       -0.2615913000     0.0000000000     0.0000000000
        0.6106597000     0.0000000000     1.0000000000
        0.5651997000     1.0000000000     0.0000000000
   P       4    2
               6.00100000
               1.99200000
               0.19480000
               0.05655000
       -0.0034030000     0.0000000000
       -0.0192089000     0.0000000000
        0.4925534000    -0.2130858000
        0.6144261000     1.0000000000
   D       1    1
               0.19330000
        1.0000000000
   ECP                     # ECP数据部分
   Al    10    2    2      # 元素符号，芯电子数，ECP最高角动量，SOECP最高角动量（可选）
   D potential  4                                    # ECP最高角动量（D函数）的项数
      2      1.22110000000000     -0.53798100000000  # R的幂，指数，因子（下同）
      2      3.36810000000000     -5.45975600000000
      2      9.75000000000000    -16.65534300000000
      1     29.26930000000000     -6.47521500000000
   S potential  5                                    # S投影的项数
      2      1.56310000000000    -56.20521300000000
      2      1.77120000000000    149.68995500000000
      2      2.06230000000000    -91.45439399999999
      1      3.35830000000000      3.72894900000000
      0      2.13000000000000      3.03799400000000
   P potential  5                                    # P投影的项数
      2      1.82310000000000     93.67560600000000
      2      2.12490000000000   -189.88896800000001
      2      2.57050000000000    110.24810400000000
      1      1.75750000000000      4.19959600000000
      0      6.76930000000000      5.00335600000000
   P so-potential  5                                 # P SO投影的项数，标量ECP没有这一部分
      2      1.82310000000000      1.51243200000000  # 标量ECP没有这一部分
      2      2.12490000000000     -2.94701800000000  # 标量ECP没有这一部分
      2      2.57050000000000      1.64525200000000  # 标量ECP没有这一部分
      1      1.75750000000000     -0.08862800000000  # 标量ECP没有这一部分
      0      6.76930000000000      0.00681600000000  # 标量ECP没有这一部分
   D so-potential  4                                 # D SO投影的项数，标量ECP没有这一部分
      2      1.22110000000000     -0.00138900000000  # 标量ECP没有这一部分
      2      3.36810000000000      0.00213300000000  # 标量ECP没有这一部分
      2      9.75000000000000      0.00397700000000  # 标量ECP没有这一部分
      1     29.26930000000000      0.03253000000000  # 标量ECP没有这一部分
   ****

对于标量ECP，SOECP最高角动量为0（可以省略不写），也不需要提供SO投影部分的数据。

把以上数据保存后，就可以在BDF输入文件中调用 ``MYBAS-1`` 基组，这需要通过以下的混合输入模式实现：

.. code-block:: bdf

    #!bdfbasis.sh
    HF/genbas 

    Geometry
     .....
    End geometry

    $Compass
    Basis
       mybas-1         # 给出当前目录下基组文件的名字，这里不区分大小写
    $End

自定义基组必须用BDF的混合模式输入。在第二行输入基组设置为 **genbas** , 自定义基组文件名需要在 **COMPASS** 模块使用关键词 ``Basis`` ，值为 ``mybas-1`` ，表示调用名为 ``MYBAS-1`` 的基组文件。

基组的指定
------------------------------------------------
**对所有原子使用相同的BDF内置基组**

简洁输入模式，基组在 ``方法/泛函/基组`` 或者 ``方法/基组`` 中指定。这里 ``基组`` 是前几节所列的BDF内置的基组名称，输入字符大小写不敏感，如下所示：

.. code-block:: bdf

   #! basisexample.sh
   TDDFT/PBE0/3-21g

   Geometry
   H   0.000   0.000    0.000
   Cl  0.000   0.000    1.400
   End geometry


.. code-block:: bdf

   #! basisexample.sh
   HF/lanl2dz 

   Geometry
   H   0.000   0.000    0.000
   Cl  0.000   0.000    1.400
   End geometry

如果是高级输入模式，计算采用的基组在 ``compass`` 模块中利用关键词 ``basis`` 指定，例如

.. code-block:: bdf

  $compass
  Basis
   lanl2dz
  Geometry
    H   0.000   0.000    0.000
    Cl  0.000   0.000    1.400
  End geometry
  $end

其中 ``lanl2dz`` 调用内置的LanL2DZ基组（已在 ``basisname`` 文件中注册），不区分大小写。

**为不同元素指定不同基组** 

简洁输入不支持自定义或者混合基组，必须采用混合输入模式，即在 ``方法/泛函/基组`` 中设置 ``基组`` 为 ``genbas`` , 并添加 **COMPASS** 模块输入，使用 ``basis-block`` ... ``end basis`` 关键词指定基组。

如果对不同元素指定不同名称的基组，需要放在 **COMPASS** 模块的 ``basis-block`` ... ``end basis`` 块中，
其中第一行是默认基组，之后的行对不同元素指定其它基组，格式为 *元素=基组名* 或者 *元素1,元素2, ...,元素n=基组名* 。

例如，混合输入模式下，对不同原子使用不同基组的示例如下：

.. code-block:: bdf

  #! multibasis.sh
  HF/genbas 

  Geometry
  H   0.000   0.000    0.000
  Cl  0.000   0.000    1.400
  End geometry

  $compass
  Basis-block
   lanl2dz
   H = 3-21g
  End Basis
  $end

上例中，H使用3-21G基组，而未额外定义的Cl采用默认的LanL2DZ基组。

如果是高级输入，如下：

.. code-block:: bdf

  $compass
  Basis-block
   lanl2dz
   H = 3-21g
  End Basis
  Geometry
    H   0.000   0.000    0.000
    Cl  0.000   0.000    1.400
  End geometry
  $end

**为同种元素的不同原子指定不同基组** 

BDF也可以为同一元素中的不同原子指定不同名称的基组，这些原子需要在元素符号后加上任意的数字以示区分。例如，


.. code-block:: bdf

  #! CH4.sh
  RKS/B3lyp/genbas

  Geometry
    C       0.000   -0.000    0.000
    H1     -0.000   -1.009   -0.357
    H2     -0.874    0.504   -0.457
    H1      0.874    0.504   -0.357
    H2      0.000    0.000    1.200
  End geometry

  $compass
  Basis-block
   6-31g
   H1= cc-pvdz
   H2= 3-21g
  End basis
  $end

上例中，H1类型的两个氢原子用cc-pVDZ基组，H2类型的两个氢原子用3-21G基组，碳原子用6-31G基组。需要注意的是，对称等价原子必须使用相同基组，程序将对此进行检查；
如果对称等价原子必须要使用不同基组，可通过 ``Group`` 设置较低的点群对称性，或者用 ``Nosymm`` 关闭对称性。

辅助基组
------------------------------------------------
使用密度拟合近似（RI）的方法需要一个辅助的基组。Ahlrichs系列基组和Dunning相关一致性基组以及其它个别基组有专门优化的辅助基组。BDF中可以在compass中通过 ``RI-J``、 ``RI-K`` 和 ``RI-C`` 关键词指定辅助基组。其中 ``RI-J`` 用于指定库伦拟合基组， ``RI-K`` 用于指定库伦交换拟合基组， ``RI-C`` 用于指定库伦相关拟合基组。BDF支持的辅助基组保存在 ``$BDFHOME/basis_library`` 路径下对应的文件夹中。

高级别密度拟合基组可以用在低级别基组上，例如 ``cc-pVTZ/C`` 可以用于在 ``cc-pVTZ`` 上做RI-J，对于没有标配辅助基组的pople系列基组如 ``6-31G**`` 也可以用 ``cc-pVTZ/J`` 做RI-J或RIJCOSX。反之，高级别轨道基组结合低级别的辅助基组则会带来较明显的误差。

.. code-block:: bdf

  $Compass
  Basis
    DEF2-SVP
  RI-J
    DEF2-SVP
  Geometry
    C          1.08411       -0.01146        0.05286
    H          2.17631       -0.01146        0.05286
    H          0.72005       -0.93609        0.50609
    H          0.72004        0.05834       -0.97451
    H          0.72004        0.84336        0.62699
  End Geometry
  $End

上例中，使用 ``def2-SVP`` 基组计算 :math:`\ce{CH4}` 甲烷分子，同时用def2-SVP标配的库伦拟合基组进行加速计算。

.. hint::
    BDF的RI计算功能，用于加速 **MCSCF**、 **MP2** 等波函数计算方法，不推荐用户在 **SCF** 、 **TDDFT** 等计算中使用，用户可以用多级展开库伦势 (MPEC) 方法，MPEC方法不依赖辅助基组，计算速度和精度都与RI方法相当。
