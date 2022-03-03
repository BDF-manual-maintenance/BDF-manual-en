
.. _Point-Group:

Symmetry and molecular point groups
================================================

BDF supports the consideration of molecular point group symmetries in the computation. 
Most computational tasks support any real representation point group（all Abelian groups, as well as :math:`\rm C_{nv}, D_{n}, D_{nh}, D_{nd}, T_d, O, O_h, I, I_h` ；
the special point groups :math:`\rm C_{\infty v}, D_{\infty h}` are nominally supported but are treated as :math:`\rm C_{20v}` and :math:`\rm D_{20h}` respectively.
except for some computational tasks (e.g., open-shell TDDFT, TDDFT/SOC, etc.) that support only :math:`\rm D_{2h}` and its subgroups (i.e.,  :math:`\rm C_1, C_i, C_s, C_2, D_2, C_{2h}, C_{2v}, D_{2h}` ，generally referred to as Abelian groups)
while individual atoms are treated as :math:`\rm O_{h}` groups), but not the complex representation point groups（ :math:`\rm C_n, C_{nh} (n \ge 3); S_{2n} (n \ge 2); T, T_h` ）。

The program can automatically determine the point group to which a molecule belongs based on the coordinates of the molecule entered by the user in COMPASS module, and automatically change to the appropriate subgroup when the molecule belongs to the complex representation point group. After determining the point group to which the molecule belongs, the program generates the group operator, the characteristic scale, and the integrable representation of the point group for subsequent calculations. Take ammonia molecule as an example.

.. code-block:: bdf

    #! NH3.sh
    HF/cc-pVDZ 

    geometry
     N                 -0.00000000   -0.00000000   -0.10000001
     H                  0.00000000   -0.94280900    0.23333324
     H                 -0.81649655    0.47140450    0.23333324
     H                  0.81649655    0.47140450    0.23333324
    end geometry

    $compass
    Title
      NH3
    thresh
      medium
    $end

The corresponding advanced input mode in **COMPASS** is

.. code-block:: bdf

  $COMPASS
  Title
   NH3
  Basis
   cc-pvdz
  Geometry
   N                 -0.00000000   -0.00000000   -0.10000001
   H                  0.00000000   -0.94280900    0.23333324
   H                 -0.81649655    0.47140450    0.23333324
   H                  0.81649655    0.47140450    0.23333324
  End geometry
  thresh
   medium 
  $END

Note that since the initial structure does not strictly satisfy the :math:`\rm C_{3v}` symmetry, the ``thresh medium`` is used here to choose a looser threshold for judging the symmetry (the default is ``tight``, or a looser one can be ``loose`` ）。As can be seen from the output file, the program automatically identifies the molecule as belonging to the :math:`\rm C_{3v}` point group.

.. code-block:: 

  gsym: C03V, noper=    6
   Exiting zgeomsort....
   Representation generated
    Point group name C(3V)                        6
    User set point group as C(3V)
    Largest Abelian Subgroup C(S)                         2

Note that the subscripts of the point group names need to be enclosed in parentheses, e.g. :math:`\rm C_{\infty v}, D_{\infty h}` groups need to be written as C(LIN), D(LIN). Next, the integrable representation information, the CG coefficient table, etc. are printed. At the end of the COMPASS section output, the program gives a list of the integrable representations under the point group and the number of orbits belonging to each integrable representation.

.. code-block:: 

  |--------------------------------------------------|
            Symmetry adapted orbital

    Total number of basis functions:      29      29

    Number of irreps:   3
    Irrep :     A1        A2        E1
    Norb  :     10         1        18
  |--------------------------------------------------|
  
Order of integrable representations
---------------------------------------------

Very often, the user needs to specify in the input file information such as the number of orbits occupied by each integrable representation (specified in the input to the SCF module) and how many excited states are calculated under each integrable representation (specified in the input to the TDDFT module), which is generally provided in the form of an array, e.g.

.. code-block:: bdf

  $TDDFT
  Nroot
   3 1 2
  $END

It indicates that the first integrability indicates the calculation of 3 excited states, the second integrability indicates the calculation of 1 excited state, and the third integrability indicates the calculation of 2 （see the :ref:`TDDFT section <TD>` of this manual for details）。This necessarily requires the user to know the order of the integrable representations inside the BDF program at the time of writing the input file. The order of the integrable representations for all point groups supported by the BDF is given below.

.. table:: The order of irreducible representations under different point groups
   :widths: 30 70

   ==================== ======================================================================================================
   C(1)                 A
   C(i)                 Ag, Au
   C(s)                 A', A"
   C(2)                 A, B
   C(2v)                A1, A2, B1, B2
   C(2h)                Ag, Bg, Au, Bu
   D(2)                 A, B1, B3, B2
   D(2h)                Ag, B1g, B3g, B2g, Au, B1u, B3u, B2u
   C(nv) (n=2k+1, k>=1) A1, A2, E1, ..., Ek
   C(nv) (n=2k+2, k>=1) A1, A2, B1, B2, E1, ..., Ek
   D(n)  (n=2k+1, k>=1) A1, A2, E1, ..., Ek
   D(n)  (n=2k+2, k>=1) A1, A2, B1, B2, E1, ..., Ek
   D(nh) (n=2k+1, k>=1) A1', A2', E1', ..., Ek', A1", A2", E1", ..., Ek", 
   D(nh) (n=2k+2, k>=1) A1g, A2g, B1g, B2g, E1g, ..., Ekg, A1u, A2u, B1u, B2u, E1u, ..., Eku
   D(nd) (n=2k+1, k>=1) A1g, A2g, E1g, ..., Ekg, A1u, A2u, E1u, ..., Eku
   D(nd) (n=2k+2, k>=1) A1', A2', B1', B2', E1', ..., Ek', A1", A2", B1", B2", E1", ..., Ek"
   T(d)                 A1, A2, E, T1, T2
   O                    A1, A2, E, T1, T2
   O(h)                 A1g, A2g, Eg, T1g, T2g, A1u, A2u, Eu, T1u, T2u
   I                    A, T1, T2, F, H
   I(h)                 Ag, T1g, T2g, Fg, Hg, Au, T1u, T2u, Fu, Hu
   ==================== ======================================================================================================

The user can also force the program to calculate under a subgroup of the point group to which the molecule belongs by using the group keyword in the COMPASS module input, e.g.

.. code-block:: bdf

  #! N2.sh
  HF/def2-TZVP group=D(2h) 

  geometry
    N  0.00 0.00 0.00
    N  0.00 0.00 1.10
  end geometry

or

.. code-block:: bdf

  $COMPASS
  Title
   N2
  Basis
   def2-TZVP
  Geometry
   N 0.00 0.00 0.00
   N 0.00 0.00 1.10
  End geometry
  Group
   D(2h)
  $END

That is, the program is forced to compute the  :math:`\rm N_2` molecule under the :math:`\rm D_{2h}` point group, even though the :math:`\rm N_2` molecule actually belongs to the :math:`\rm D_{\infty h}` point group. Note that the program automatically checks whether the point group entered by the user is a subgroup of the point group to which the molecule actually belongs; if not, the program quits with an error.

Standard orientation
---------------------------------------------

For the convenience of the calculation and the analysis of the results, the program rotates the molecules to the standard orientation after determining the point group used for the calculation, so that the symmetry axes of the molecules coincide as much as possible with the coordinate axes and the symmetry plane is as perpendicular as possible to the coordinate axes. This has the advantage that many quantities involved in the calculation are exactly equal to zero (e.g., some molecular orbital coefficients, some components of the gradient, etc.), which makes it easier to analyze the results.

The BDF determines the standard orientation of a molecule according to the following rules.

1. take a weighted average of all atomic coordinates of the molecule by nuclear charge to obtain the nuclear charge center of the molecule, and then translate the molecule so that the nuclear charge center is at the origin of the coordinate system.
2. if the molecule has an axis of symmetry, rotate the highest order axis of symmetry (principal axis) of the molecule to the z-axis direction.
3. if the molecule has :math:`\sigma_v` symmetry planes, rotate one of the :math:`\sigma_v` symmetry planes to the xz-plane direction, keeping the major axis direction constant in the process.
4. if the molecule has other dual or quadruple axes besides the principal axes, rotate one of the axes (any of the quadruple axes if they exist, otherwise any of the dual axes) to the x-axis direction, keeping the direction of the principal axes unchanged in the process.
5. if the above conditions cannot uniquely determine the orientation of the molecule because the symmetry of the molecule is too low, rotate the molecule so that the axis of inertia (i.e., the eigenvector of rotational inertia) of the molecule and each coordinate axis are oriented in the same direction.

For some special cases, the above rule still cannot uniquely determine the orientation of the molecule. For example, molecules belonging to the :math:`\rm C_{2v}` point group have two :math:`\sigma_v` symmetry planes, and either of them may be rotated to the xz direction at step 3 of the above rule. In the BDF, a :math:`\rm C_{2v}` molecule with a planar structure, such as a water molecule, will be rotated to the xz plane.

.. code:: bdf

  |-----------------------------------------------------------------------------------|

   Atom   Cartcoord(Bohr)               Charge Basis Auxbas Uatom Nstab Alink  Mass
    O   0.000000  -0.000000   0.219474   8.00   1     0     0     0   E     15.9949
    H  -1.538455   0.000000  -0.877896   1.00   2     0     0     0   E      1.0073
    H   1.538455  -0.000000  -0.877896   1.00   2     0     0     0   E      1.0073

  |------------------------------------------------------------------------------------|

In contrast, other quantization programs may choose to rotate the numerator to the yz plane. 
This leads to another problem: according to the customary convention, the  :math:`\mathbf{x}` operator under the :math:`\rm C_{2v}` point group belongs to the B1 integrable representation and the :math:`\mathbf{y}` operator belongs to the B2 integrable representation,
so if a quantization program chooses to rotate the numerator to the yz plane, its B1 and B2 integrable representations are defined opposite to the BDF,
i.e., the B1 representation of the program corresponds to the B2 representation of the BDF, and the B2 representation of the program corresponds to the B2 representation of the BDF of the B1 representation of the BDF.
And if the molecule of this :math:`\rm C_{2v}` point group is not a planar structure (e.g., ethylene oxide), it is even more difficult to predict whether the standard orientation of the molecule in the BDF is consistent with other quantization software. 
Therefore, if the user wishes to calculate the molecules of the :math:`\rm C_{2v}` point group and compare the results with those of other quantization programs (or try to duplicate the results calculated by other quantization programs in the literature), the user must confirm how the B1 and B2 representations of the quantization program correspond to the BDF.








