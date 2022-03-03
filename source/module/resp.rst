DFT/TDDFT Gradient and Response Properties - RESP Module
==========================================================
The resp module is used to calculate the gradient of the DFT/TDDFT, the non-adiabatic coupling matrix elements at the TDDFT theoretical level (including the non-adiabatic coupling matrix elements between the ground-state-excited state, and the non-adiabatic coupling matrix elements between the excited state-excited state), and the response properties such as the excited state dipole moment.

**Basic Keywords**

:guilabel:`Iprt` parameter type: integer
------------------------------------------------
Controls the printout level, mainly used for program debugging.

:guilabel:`NOrder` parameter type: integer
------------------------------------------------
 * Default value：1
 * Optional values：0、1、2

The order of the geometric derivative is currently supported0 only for (response properties that do not involve analytic gradients, such as excited state dipole moments) and1 (analytic gradients), but not yet for 2(analytic Hessian). This parameter requires that Geom be specified first.

:guilabel:`Geom` parameter type: Bool type
------------------------------------------------
This keyword does not require parameters and needs to be used in conjunction with the Norder keyword to specify the first or second order derivative of the calculated geometric coordinates.

ptional values: 1. Gradient or fo-NACMEs; 2. Hessian (under development)


:guilabel:`NFiles` parameter type: integer
------------------------------------------------
For TD-DFT response property calculations, specify which $tddft block to read; note that when this parameter is equal to x, it does not simply mean that the xth $tddft block is read, but that the $tddft block with an istore value of x is read. For example, the following input for a closed-shell molecule ($compass, $xuanyuan, $scf omitted)

.. code-block:: bdf

     $tddft
     imethod
     1
     Nroot
     1
     istore
     1
     $end

     $tddft
     imethod
     1
     isf
     1
     Nroot
     1
     istore
     2
     $end

     $resp
     geom
     imethod
     2
     nfiles
     2            #Calculate the gradient of the lowest triple excited state, not the gradient of the lowest single excited state
                  #Because nfiles=2, only the second $tddft block (lowest triple excitation state) isore=2
     $end

:guilabel:`Imethod` parameter type: integer
------------------------------------------------
 * Default value：1
 * Optional values：1、2

Specifies whether to perform DFT ground state or TD-DFT excited state calculations. 1 is the ground state, if specified2 , then it is the excited state calculation. In older versions of BDF the keyword is written Method, currently the program supports both Imethod and Method, but in the future it may only support the former.

.. code-block:: bdf

     #Calculate the TD-DFT gradient of the first TD-DFT excited state
     $tddft
     Nroot
     1
     istore
     1
     $end

     $resp
     geom
     imethod
     2
     nfiles
     1
     $end

.. code-block:: bdf

     #Calculate the ground-state gradient
     $resp
     geom
     $end

:guilabel:`Ignore` parameter type: integer
------------------------------------------------
 * Default value：0
 * Optional values：-1、0、1

Data consistency check for TDDFT gradient calculation, mainly for debugging programs.

-1：Recalculates the TDDFT excitation energy, used to check that the Resp and TDDFT modules agree on the energy calculation. For debugger use only.

0: Check if the Wmo matrix is a symmetric matrix. Theoretically, the Wmo matrix should be symmetric, but if the TDDFT or Z-Vector iterations do not converge completely, the Wmo matrix will show significant asymmetry, and the program will exit with an error and tell the user whether the Wmo matrix is asymmetric because the TDDFT did not converge completely or the Z-Vector equation did not converge completely. Note that sometimes the asymmetry of the Wmo matrix can also be caused by some keyword input errors by the user.

1: Ignore the Wmo matrix symmetry check. The ignore setting should only be1 set if the user has confirmed that the TDDFT and Z-vector convergence thresholds are tight enough to not affect the accuracy of the computation unacceptably, and that the keywords in the input file have been entered correctly, but the program still reports an error due to a failed symmetry check， the ignoore should be set to 1.

:guilabel:`IRep` & :guilabel:`IRoot` parameter type: integer
-----------------------------------------------------
These two keywords specify which/which state(s) of TD-DFT gradient or excited state dipole moment is/are to be calculated. There are 4 cases：

a.	Specify both IRep and IRoot: e.g. the following input

.. code-block:: bdf

     #Calculates the gradient or dipole moment of the 3rd root under the 2nd irreducible representation (irrep).
     irep
     2
     iroot
     3

b.	Specify IRep only: Compute the gradient or dipole moment of all roots under this integrable representation.

c.	Specify IRoot only: for example

.. code-block:: bdf

     #All the roots under irreducible representation are sorted by energy from low to high, and then the gradient or dipole moment of the 3rd root is calculated
     iroot
     3
     
d.	Neither is specified: calculate the gradient or dipole moment of all states obtained by tddft.

:guilabel:`JahnTeller` Parameter type: String
------------------------------------------------
For molecules with certain symmetries, TDDFT structure optimization may lead to Jahn-Teller distortion of the molecule if the point group to which the molecule belongs is a higher-order point group, but the distortion may have multiple directions. For example, assuming that a molecule with Ih symmetry has a triple-simplex excited state T2g, the symmetry of the geometry of this state may be reduced to D2h, D3d, D5d or subgroups of these groups after the Jahn-Teller distortion. Therefore, in the TDDFT structure optimization, the symmetry of the molecular structure may decrease from the second optimization step. When the point group obtained by Jahn-Teller distortion is not unique, the specific Jahn-Teller distortion can be specified by the JahnTeller keyword. For example，

.. code-block:: bdf

     $resp
     ...
     JahnTeller
      D(2h)
     $End
   
The above example specifies that when there is a Jahn-Teller aberration and the aberration mode is not unique, preference is given to the aberration mode in which the aberrated structure belongs to the D2h group. If it can be deduced from group theory that the molecule will not undergo a Jahn-Teller aberration in the current electronic state, or that a Jahn-Teller aberration will occur but will not result in a structure belonging to the D2h group, the program prints a warning message and ignores the user input. If the current molecule will undergo a Jahn-Teller distortion but the user does not specify a JahnTeller keyword, the program tries to maintain the higher order symmetry axis of the molecule during the Jahn-Teller distortion. Still using the T2g state of the Ih group above as an example, if the JahnTeller keyword is not specified, the molecule will distort to a D5d structure because this is the only way to maintain the fivefold symmetry axis of the Ih group.

:guilabel:`Line` parameter type: Bool type
------------------------------------------------
Perform resp for linear response calculation.

:guilabel:`Quad` parameter type: Bool type
------------------------------------------------
Specify resp for secondary response calculation.

:guilabel:`Fnac` parameter type: Bool type
------------------------------------------------
Specify resp to calculate the first-oder noadibatic couplings vectors, which need to be used in conjunction with the Single or Double parameters to specify the calculation of the ground-state-excited state and excited-state-excited state noadibatic couplings vectors, respectively.

:guilabel:`Single` parameter type: Bool type
------------------------------------------------
Specify the calculation of the ground-state-excited state non-adiabatic coupling vector.

:guilabel:`States` parameter type: integer数组
------------------------------------------------
Specifies which states are calculated for the non-adiabatic coupling vector to the ground state. This parameter is a multi-line parameter.

First line: Enter the integer n, specifying the non-adiabatic coupling vector between the ground state and the n excited states to be calculated.

The second line to line n+1 specifies the electronic state in the format of three integers m i l. m is the file number of the previous TDDFT calculation istore specified storage, i is the i-th integrable representation, and l is the l-th root of that integrable representation.


:guilabel:`Double` parameter type: Bool type
------------------------------------------------
Specify the excited-state excited-state non-adiabatic coupling vector for calculation.

:guilabel:`Pairs` parameter type: integer数组
------------------------------------------------
Specifies which set of two excited states to calculate the non-adiabatic coupling vector between. This parameter is a multi-line parameter：

First line: Enter an integer n, specifying that the non-adiabatic coupling vector between n pairs of excited states is to be calculated.

The second to n+1 lines specify the electronic states in the format m1 i1 l1 m2 i2 l2 six integers, with each three integers specifying an excited state. m1 is the file number of the storage specified by the previous TDDFT calculation istore, i1 is the i1st integrable representation, and l1 is the l1st root of that integrable representation. The other three integers are the same.

:guilabel:`Noresp` parameter type: Bool type
------------------------------------------------
Specifies that the response term of the leap density matrix is ignored in Double and FNAC calculations. This keyword is recommended.

