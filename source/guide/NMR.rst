
Nuclear magnetic resonance shielding constants
================================================

BDF supports the restricted Hartree-Fock (RHF) and restricted Kohn-Sham (RKS) methods for NMR shielding constant (NMR) calculations, where the problem of the outer vector potential canonical origin can be handled using the Common gauge method and GIAO (gauge-including atomic orbitals).

.. warning::

    Since the libcint library is required for NMR calculation, a line needs to be added in the calculation script: export USE_LIBCINT=yes



NMR calculation example
----------------------------------------------------------
The following is the input file for the NMR shielding constant calculation of methane molecules：


.. code-block:: bdf

  $COMPASS  # Molecular coordinate input and symmetry judgment
  Title
  CH4 Molecule NR-DFT-NMR       # job title
  Basis
  CC-PVQZ                       # basis set
  Geometry
  C  0.000   00000    0.000     # molecule geometry
  H  1.196   1.196    1.196
  H -1.196  -1.196    1.196
  H -1.196   1.196   -1.196
  H  1.196  -1.196   -1.196
  END geometry
  nosymm                        # NMR module does not support symmetry 
  UNIT
    BOHR                        # input molecule geometry in bohr
  $END

  $xuanyuan # Single and double electron integral correlation setting and calculation
  $end

  $SCF      # Self consistent field calculation module
  RKS       # Restrict Kohn-Sham
  DFT
    b3lyp
  $END

  $NMR      # NMR shielding constant calculation module
  icg
   1        # can enter 0 or 1. 0 means no common gauge calculation and 1 means common gauge calculation. The default value is 0.
  igiao
   1        # can enter 0 or 1. 0 means no GIAO calculation and 1 means GIAO calculation. The default value is 0.
  $END


The four modules ``compass`` ,  ``xuanyuan`` , ``scf`` and ``nmr`` are called sequentially to complete the calculation. The ``scf`` module performs the ``RKS`` calculation. Based on the results of the RKS calculation, subsequent ``NMR`` calculations are performed, where the ``NMR`` calculation is followed by the COMMON GAUGE calculation and the GIAO calculation, which gives the isotropic and anisotropic NMR shielding constants for all atoms.


COMMON GAUGE
----------------------------------------------------------
The NMR calculation of COMMON GAUGE can be controlled by the keyword icg：

.. code-block:: bdf 

  $NMR
  icg
    1
  $END

You can enter 0 or 1. The default value is 0, which means that no COMMON GAUGE calculation is performed, and 1, which means that a COMMON GAUGE calculation is performed.

In the COMMON GAUGE calculation, the canonical origin is located at the origin of the coordinates (0, 0, 0) by default. The canonical origin can be specified at an atom by using the keyword igatom, or it can be set to a specified location in space by using cgcoord, as follows.：

.. code-block:: bdf 

  $NMR
  icg
    1
  igatom
    3             # Specify the gauge origin on atom 3, enter an integer number, ranging from 0 to the number of molecular atoms，
                  # If you enter a value of 0, the specification origin is specified at the coordinate origin
  cgcoord
    1.0 0.0 0.0   # enter 3 real numbers, and place the origin of the specification on the point with the spatial coordinates of (1.0, 0.0, 0.0)
  cgunit
    angstrom      # The unit of cgcoord coordinate. The default value is atomic unit. When the input is Angstrom, the input standard origin coordinate
                  # In angstroms; For other inputs (such as Bohr, AU), the coordinate unit is atomic unit, and the input is case insensitive
  $END

When both igatom and cgcoord are present in the input, the latter input prevails. For example, in the above example, the final canonical origin is set at the spatial coordinates (1.0, 0.0, 0.0) (in angstroms). If both parameters igatom and cgcoord are not entered, the NMR value of COMMON GAUGE is calculated with the normative origin at the coordinate origin, i.e. at the position (0.0, 0.0, 0.0)

The Common gauge calculation in the output file starts at ``[nmr_nr_cg]`` , as follows：

.. code-block:: bdf 

  [nmr_nr_cg]
    Doing nonrelativistic-CG-DFT nmr...

  [nmr_set_common_gauge]
    set the common gauge origin as the coordinate origin(default)
        0.000000000000      0.000000000000      0.000000000000

Omitting the middle part of the output, the final result is output as follows：

.. code-block:: bdf 

  Isotropic/anisotropic constant by atom type:
    atom-C
      186.194036      0.000003
    atom-H
       31.028177      9.317141
       31.028176      9.317141
       31.028177      9.317141
       31.028177      9.317141

The NMR shielding constants in ppm for C and H atoms, respectively, are shown in the first column as isotropic shielding constants and in the second column as anisotropic shielding constants.


GIAO
----------------------------------------------------------
The NMR calculation of GIAO can be controlled by the keyword igiao：

.. code-block:: bdf 

  $NMR
  igiao
    1
  $END

You can enter either 0 or 1. The default value is 0, i.e., no GIAO calculation is performed, and when entered as 1, GIAO calculation is performed.

.. warning::
  In the NMR module, icg and igiao can be entered as either 1, which sets either calculation to be performed, or both (i.e. both calculations), but not both or 0, otherwise the NMR module will not produce any NMR shielding constant values.

The GIAO calculation in the output file starts at ``[nmr_nr_giao]`` , as follows：

.. code-block:: bdf

 [nmr_nr_giao]
  Doing nonrelativistic-GIAO-DFT nmr

 [set_para_for_giao_eri]

 [nmr_int]
   Doing nmr integral of operators resulting from the response of B10...

   No. of pGTOs and cGTOs:     196     196

   giao integrals...

Omitting the middle part of the output, the final result is output as follows：

.. code-block:: bdf 

    Isotropic/anisotropic constant by atom type:
      atom-C
        186.461988      0.000019
      atom-H
        31.204947      9.070916
        31.204944      9.070916
        31.204947      9.070921
        31.204946      9.070920

As in the case of COMMON GAUGE, the above results are the GIAO shielding constants in ppm for C and H atoms, respectively, with the first column being the isotropic shielding constant and the second column being the anisotropic shielding constant。

.. warning::
  The keyword ``Isotropic/anisotropic constant by atom type`` in the output is exactly the same for GIAO and COMMON GAUGE, when reading the result, you should pay attention to whether it is after ``[nmr_nr_cg]`` or after ``[nmr_nr_giao]`` to distinguish between COMMON GAUGE result or GIAO result.