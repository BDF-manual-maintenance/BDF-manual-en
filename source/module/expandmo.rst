Different Basis Set Expansion Tracks - EXPANDMO Module
============================================================
The EXPANDMO module is used to extend the MO of a small basis group calculation to a large basis set MO. The extended MO can be used for the initial guesses of the SCF and also for some Dual Basis calculations. In addition, EXPANDMO can automatically construct the active space and initial guess orbits for MCSCF calculations using the atomic valence active space.

:guilabel:`Overlap` parameter type: Bool type
------------------------------------------------
Specify the expansion of molecular orbitals using the overlapping integrals of the small and large basis groups.

The Expandmo module dependency file is as follows：

+--------------------+---------------------------------------------------------+-------------+----- -----+
| filename           | description                                             | file format |           |
+--------------------+---------------------------------------------------------+-------------+-----------+
| $BDFTASK.chkfil1   | The check file for the small basis set calculation      | Binary      | input file|
+--------------------+---------------------------------------------------------+-------------+-----------+
| $BDFTASK.chkfil2   | The check file for the big basis set calculation        | Binary      | input file|
+--------------------+---------------------------------------------------------+-------------+-----------+
| inporb             | The MO file produced by the small basis set calculation | text file   | input file|
+--------------------+---------------------------------------------------------+-------------+-----------+
| $BDFTASK.exporb    | The extended MO coefficient file is stored in           | text file   | input file|
|                    | BDF_WORKDDIR中                                          |             |           |
+--------------------+---------------------------------------------------------+-------------+-----------+

.. code-block:: bdf

     #Calculate CH2 molecules with cc-pVDZ basis sets and extend molecular orbital coefficients to aug-cc-pVDZ group for initial guessing of SCF calculations
     # First we perform a small basis set calculation by using CC-PVDZ.
     $COMPASS
     Title
       CH2 Molecule test run, cc-pvdz
     Basis
       cc-pvdz
     Geometry
     C     0.000000        0.00000        0.31399
     H     0.000000       -1.65723       -0.94197
     H     0.000000        1.65723       -0.94197
     End geometry
     UNIT
       Bohr
     Check
     $END

     $XUANYUAN
     $END

     $SCF
     RHF
     Occupied
     3  0  1  0
     $END

     #Change the name of check file.
     %mv $BDF_WORKDIR/ch2.chkfil $BDF_WORKDIR/ch2.chkfil1
     #Copy converged SCF orbital to work directory inporb.
     %mv $BDF_WORKDIR/ch2.scforb $BDF_WORKDIR/ch2.inporb

     # Then we init a large basis set calculation by using aug-CC-PVDZ
     $COMPASS
     Title
       CH2 Molecule test run, aug-cc-pvdz
     Basis
       aug-cc-pvdz
     Geometry
     C     0.000000        0.00000        0.31399
     H     0.000000       -1.65723       -0.94197
     H     0.000000        1.65723       -0.94197
     End geometry
     UNIT
       Bohr
     Check
     $END
