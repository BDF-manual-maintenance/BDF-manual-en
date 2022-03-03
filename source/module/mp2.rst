Møller–Plesset Second Order Perturbation - MP2 Module
=======================================================
Møller-Plesset second-order perturbation theory calculation module, mainly for implementing two-hybrid DFT calculations. 
MP2 supports both RHF and UHF reference wavefunctions for the integral undirected symmetric matched-orbit SCF-based algorithm (see :ref:`Saorb<compass.saorb>`  keyword) and only RHF reference wavefunctions for the integral direct SCF algorithm (by default, see  :ref:`Skeleton<compass.skeleton>`  keyword).


:guilabel:`Nature` parameter type: Bool type
------------------------------------------------
Calculate the approximate density matrix and output the natural orbit.

:guilabel:`Molden` parameter type: Bool type
---------------------------------------------------
Output natural tracks as molden format files.

:guilabel:`Iprtmo` parameter type: Integer
------------------------------------------------
Controls the track output print mode.

:guilabel:`Fss, Fos` parameter type: floating point
------------------------------------------------
 * Default value：1.0

SCS-MP2 and the spin component scaling parameter used in some two-hybrid generalized functions. After calculating the MP2 energy, the program multiplies the same-spin component by Fss and the opposite-spin component by Fos.
