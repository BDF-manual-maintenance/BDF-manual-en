Energy and Charge Transfer - ELECOUP Module
================================================
Energy and Charge Transfer - ELECOUP Module：

#. Calculating the coupling integral between two electronic states of the same molecule based on HF； 
#. Calculate the charge migration integral between two molecular sheets； 
#. Calculate the energy migration integral between the excited states of two molecular sheets.

.. note::

    **The HF-based calculation of the coupling integral between two excited states of the same molecule requires the use of the ΔSCF method, usually using the mom function in the SCF module.**

:guilabel:`Iprt` parameter type: integer
------------------------------------------------
Print control parameters for debugging programs only.

:guilabel:`UHF` parameter type: Bool type
------------------------------------------------
The coupling integral between the two electronic states is based on the UHF wave function.

:guilabel:`Nexcit` parameter type: integer
------------------------------------------------
Specify the number of excited states per molecule.

:guilabel:`GSApr` parameter type: Bool type
------------------------------------------------
Specifies whether to approximate the base state processing.

**Calculate charge migration integral keywords**

:guilabel:`Electrans` parameter type: integer array
----------------------------------------------------
This parameter is a multi-line parameter that specifies a number of pairs of Donor and Acceptor molecular orbitals and calculates the charge migration integral between them. Format as follows.

First line: enter the integer n, specifying that the migration integral between n pairs of orbits is to be calculated

Line 2 to line n+1: input three integers i j k, i is the i-th track of Donor, j is the j-th track of acceptor, and parameter k is 1 or 2, specifying alpha or beta tracks, respectively.

:guilabel:`Dft` parameter type: String
------------------------------------------------
Specifies what exchange-dependent general function is used to calculate the charge migration integral. If you do not enter this parameter, the same functional is used by default as when Kohr-Sham is calculated.

**Localized excited states**

:guilabel:`locales` parameter type: integer
------------------------------------------------
Specify the method to obtain the excited state of the localization.

 * Default value： 0  
 * Optional values： 0， 1； 0 Boys definite domain method， 1 Ruedenberg localization methods
