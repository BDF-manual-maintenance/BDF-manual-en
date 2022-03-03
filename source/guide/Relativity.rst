
.. _relativity:

Relativistic effects
================================================
The relativistic effect mainly includes two parts: scalar relativistic effect and spin-orbit coupling effect. Relativistic effects are important in the study of organic light-emitting mechanisms, such as inter-system scattering of electronic states due to spin-track coupling, and changes in the spin of transition states and products. The innermost core electrons of the atom are most strongly affected by relativistic effects on the one hand, and insensitive to chemical changes on the other hand, so there are different ways to deal with them, mainly all-electron relativistic and effective core potential (ECP) two types of methods.

1. **All-electron methods** all the inner layer electrons are made variational treatment. The common all-electron relativistic Hamiltonians are: the zero-order regular approximation (ZORA), the second-order Douglas-Kroll-Hess (DKH2), the exact dichotomy (X2C) proposed by Wenjian Liu et al. ZORA and DKH2 have no advantages in terms of accuracy and efficiency and are not recommended.

2. The core electrons of **ECP** heavy atoms are replaced by a pre-fitted effective potential function. When there are more heavy atoms, ECP can significantly reduce the variational degrees of freedom and improve the computational efficiency. The ECP is divided into two categories, pseudopotential (PP) and model core potential (MCP), depending on whether the valence electron wave function has nodes in the core layer or not, where MCP has to combine a large number of Gaussian functions in order to reproduce the valence electron wave function nodes correctly, leading to an insignificant improvement in computational efficiency, and is therefore less used. ECP in BDF programs refers to PP.

The BDF basis set library provides a large number of :ref:`all-electron relativistic basis sets<all-e-bas>` and :ref:`ECP basis sets<ecp-bas>` .


.. warning::

    1. Do not mix X2C Hamiltonian and ECP basis sets
    2. X2C relativistic calculations must use non-shrinking basis sets or specially optimized shrinking basis sets, but the first 18 elements are not mandatory.


Scalar relativistic effects
------------------------------------------------

* **All-electron methods**

The BDF can consider scalar relativistic effects through the spinless X2C Hamiltonian (sf-X2C) and its local approximation variants sf-X2C-AXR, sf-X2C-AU. For example：

.. code-block:: bdf

  $xuanyuan
  heff
   23
  nuclear
    1
  $end


In the above input, ``heff`` calls scalar relativistic Hamiltonians such as sf-X2C (3, 4, or 21), sf-X2C-AXR (atomic X-matrix approximation, 22), and sf-X2C-AU (atomic
U-transformation approximation, 23), where 21, 22, 23 have analytic derivatives. For molecular systems not involving heavy element-heavy element bonding above 5d,
sf-X2C-AU has the highest efficiency without loss of precision and is the recommended method. Otherwise, sf-X2C-AXR or sf-X2C is used.

.. _finite-nuclear:

The ``nuclear`` option set to 1 indicates the use of a finite nuclear model, which is generally not required. However, some relativistic shrinkage basis groups take
into account nucleus size effects, or when calculating the electronic properties near the nucleus, in these cases the finite nuclear model is used.

The types of calculations supported by sf-X2C and its local variants are: single point energies, structure optimization, and some :ref:`first-order single electron
properties<1e-prop>` . Analytic Hessian and second-order single-electron properties are under development.


* **ECP**

ECP must be combined with a non-relativistic Hamiltonian, where relativistic effects are implicit in the pseudopotential parameters.
The types of ECP calculations supported by the BDF are:  single-point energy and some :ref:`single electron properties<1e-prop>` . Gradient and Hessian for ECP are under development.

Spin-orbit coupling interactions
------------------------------------------------
BDF can handle spin-orbit coupling between different spin multiplet electronic states in TDDFT single-point calculations by means of the state interaction (SI)
method. It needs to be specified in the ``xuanyuan`` module via the ``hsoc`` keyword how to calculate the spin-orbit integrals. See the TDDFT section for an example.

Depending on the adopted Hamiltonian, spin-orbit coupling can also be divided into two categories: all-electron and ECP.

* **All-electron methods**

Although the contribution of the two-electron spin-orbit integral is smaller than that of the single-electron spin-orbit integral, the effect on the spin-orbit coupling effect may reach 1/5~1/3, and therefore cannot be neglected. A singleelectron spin-orbit integral + a molecular mean-field two-electron spin-orbit
integral with a single-center approximation （so1e + SOMF-1c； ``hsoc`` = 2）is suggested. It can be combined with the sf-X2C scalar relativistic Hamiltonian and, for the light element system, with the non-relativistic Hamiltonian. In addition, it is possible to correct the single-electron spin-orbit integrals for shielded nuclei :cite:`snso2000,msnso2013` or effective nuclear charges :cite:`zeff1995` , approximating the effect of the twoelectron spin-orbit integrals, but this may cause unpredictable errors on the
core-layer orbital properties.

.. _so1e-zeff:

* **ECP**

It includes two treatments：

1. the spin-orbit coupling pseudopotential, which requires the addition of an additional SO potential function to the scalar ECP (SOECP; see the :ref:`spin-orbit coupling pseudopotential basis set<soecp-bas>`  in the basis set library)
2. the effective nuclear charge :cite:`zeff1995,zeff1998` 。

Since the effect of the two-electron spin-orbit interaction is already included in the fitting parameters of the SO potential or in the empirical parameters of the effective nuclear charge,
it is sufficient to calculate the single-electron spin-orbit integral. In the BDF, the atoms described by SOECP and the atoms described by scalar ECP or all-electron non-relativity can be used separately, by setting ``hsoc`` to 10 in the :ref:`xuanyuan<xuanyuan>` module. 


It is important to note that the effective nuclear charge supports a limited number of elemental and base group types. For all-electron base groups, only the main group elements before Xe are supported, with the exception of the heavier noble gas elements Ne, Ar, and Kr. For scalar ECP base groups, although more elements are supported, the number of core electrons must be consistent, see the following table:

.. table:: The number of electrons and atoms in the core of the scalar ECP base group supported by the effective nuclear charge
    :widths: auto

    +-----------------------------+----------------------------------------+-------+
    | Atom                        | ZA                                     | NCore |
    +=============================+========================================+=======+
    | Li-F                        | 3- 9                                   | 2     |
    +-----------------------------+----------------------------------------+-------+
    | Na-Cl, Sc-Cu, Zn, Ga        | 11-17, 21-29, 30, 31                   | 10    |
    +-----------------------------+----------------------------------------+-------+
    | K -Ca                       | 19-20                                  | 18    |
    +-----------------------------+----------------------------------------+-------+
    | Ge-Br, Y -Ag, Cd, In        | 32-35, 39-47, 48, 49                   | 28    |
    +-----------------------------+----------------------------------------+-------+
    | Rb-Sr                       | 37-38                                  | 36    |
    +-----------------------------+----------------------------------------+-------+
    | Sn-I, La                    | 50-53, 57                              | 46    |
    +-----------------------------+----------------------------------------+-------+
    | Cs-Ba                       | 55-56                                  | 54    |
    +-----------------------------+----------------------------------------+-------+
    | Hf-Au, Hg, Tl               | 72-79, 80, 81                          | 60    |
    +-----------------------------+----------------------------------------+-------+
    | Pb-At                       | 82-85                                  | 78    |
    +-----------------------------+----------------------------------------+-------+

For more details, such as Zeff parameters, references, etc., see soint_util/zefflib.F90. If the effective nuclear charge method is used for unsupported elements or groups, the results of the spin-orbit coupling calculations are unreliable.

