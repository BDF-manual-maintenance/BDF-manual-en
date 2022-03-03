Nuclear magnetic shielding constant calculation - NMR module
==============================================================

NMR is used to calculate the nuclear magnetic shielding constants of molecules.

:guilabel:`igiao` parameter type: Integer
------------------------------------------------
Specify whether to calculate the NMR shielding constant for GIAO, either 0(do not do GIAO calculations) or 1 (do GIAO calculations), the default is 0 and the input form is as follows.

.. code-block:: bdf

  igiao
    1

:guilabel:`icg` parameter type: Integer
------------------------------------------------
Specify whether to calculate the NMR shielding constant for COMMON GAUGE (CG), either 0(do not do CG calculations) or1 (do CG calculations), with the default being 0, entered in the following form.

.. code-block:: bdf

  icg
    1

:guilabel:`igatom` parameter type: Integer
------------------------------------------------
Specifies the position of the canonical origin of the COMMON GAUGE calculation. The accepted input is either 0, or a value from 1 to the number of atoms, with the default being 0. When igatom is 0, the canonical origin of COMMON GAUGE is set at the origin of the spatial coordinates, while when igatom is a value from 1 to the number of atoms in the molecule, the canonical origin is set at the atomic center of the first igatom. The input form is as follows.
.. code-block:: bdf

  igatom
    3      # Sets the specification origin at the center of the 3rd atom


:guilabel:`cgcoord` parameter type: real 3 numbers
------------------------------------------------
Specifies the position of the canonical origin of the COMMON GAUGE calculation to a coordinate point in space. The default units for the input coordinates are atomic units (i.e. bohr, AU), whose units can be controlled by the parameter ``cgunit`` .

.. code-block:: bdf

  cgcoord
    1.0 0.0 0.0   # Enter 3 real numbers and place the canonical origin on a point with spatial coordinates of (1.0, 0.0, 0.0).

:guilabel:`cgunit` parameter type: string
------------------------------------------------
Given the units of the ``cgcoord`` parameter, the default is atomic units (i.e. bohr, AU), which can be changed to angstrom by entering angstrom. For other inputs (e.g. bohr, AU, etc.), the coordinates are in atomic units.

.. code-block:: bdf

  cgunit
    angstrom      # The units of the cgcoord coordinates, which default to atomic units, and when the input is angstrom, the canonical origin coordinate units are angstroms
                  # Other inputs (e.g. bohr, AU) have coordinate units in atomic units and inputs are not case-sensitive



