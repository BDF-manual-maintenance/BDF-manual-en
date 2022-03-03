Molecular orbital localization - LOCALMO module
================================================
The LOCALMO module is used to generate delocalized molecular orbitals and contains methods such as Boys, Pipek-Mezey, and a modified Boys delocalization. LOCALMO is also used to generate the initial molecular slice delocalized orbitals for the FLMO method.

**Basic control parameters**

:guilabel:`Boys` parameter type: Bool type
------------------------------------------------
Specifies to use the Boys delocalization method to delocalize the track. boys is the default method for the LOCALMO module.

:guilabel:`Mboys` parameter type: Integer type
------------------------------------------------
Specifies the use of the improved Boys delocalization method, the next row is an integer, and the exponential factor for the improved Boys method is specified.

:guilabel:`Pipek` parameter type: Bool type
------------------------------------------------
Specifies to use the Pipek-Mezey delocalization method. The default is Mulliken charge, if the Lowdin parameter is set, then the Pipek-Mezey method uses Lowdin charge instead of the default Mulliken charge. This method defaults to the Jacobi rotation domainization track, if you need to specify the Trust-Region method, you need to use the keyword Trust.

:guilabel:`Mulliken` parameter type: Bool type
------------------------------------------------
Specifies that the Pipek-Mezey method uses Mulliken charges. Default option.

:guilabel:`Lowdin` parameter type: Bool type
------------------------------------------------
The parameter specifies that the Pipek-Mezey method uses the Lowdin charge.

:guilabel:`Jacobi` parameter type: Bool type
------------------------------------------------
Specifies that the Pipek-Mezey method utilizes Jacobi rotational fixed-domain orbits.

:guilabel:`Trust` parameter type: Bool type
------------------------------------------------
Specifies that the Pipek-Mezey method utilizes the Trust Region method for domaining tracks.

:guilabel:`Hybridboys` parameter type: Integer type
------------------------------------------------
Selectable values：-100、100

Specifies that the Pipek-Mezey or Boys localization method mixes Jacobi rotation with the Trust Region method to localize the track. By default, the hybrid method is not used, if this parameter is added, the next input line must be an integer.
-100: Converts the virtual orbit to the Trust Region method to continue domaining only after the virtual orbit has first been domained 100 times with Jacobi rotation or after the domaining has reached the convergence threshold Hybridthre.
100: Converts the occupied and virtual orbit to the Trust Region method to continue domaining after the occupied orbit has first been domained 100 times with Jacobi rotation or after the domaining has reached the convergence threshold Hybridthre.

:guilabel:`Hybridthre` parameter type: floating point
------------------------------------------------
Specifies the conversion threshold for the hybrid localization method.

:guilabel:`Thresh` parameter type: floating point
------------------------------------------------
Specify the threshold for the convergence of the fixed-domain method, the input is two floating-point numbers.

:guilabel:`Tailcut` parameter type: floating point
------------------------------------------------
 * Default value: 1.D-2

Specifies the threshold value for ignoring FLMO tails.

:guilabel:`Threshpop` parameter type: floating point
------------------------------------------------
 * Default value: 1.D-1

Specifies the threshold value for Lowdin placement.

:guilabel:`Maxcycle` parameter type: Integer type
------------------------------------------------
Specifies the maximum number of cycles allowed for Boys domainization.

:guilabel:`Rohfloc` parameter type: Bool type
------------------------------------------------
Specify the localized ROHF/ROKS orbit.

:guilabel:`orbital` parameter type: string
------------------------------------------------
Specifies that the file is read into the molecular track.

.. code-block:: bdf

     $LocalMO
     Orbital
     hforb       # Specifies the hforb read-in track from scaf computing storage
     $End

:guilabel:`Orbread` parameter type: Bool type
------------------------------------------------
Specifies that the molecular tracks are read from the text file inporb in **BDF_TMPDIR**.

:guilabel:`Flmo` parameter type: Bool type
------------------------------------------------
Specifies the projection LMO to pFLMO.

:guilabel:`Frozocc` parameter type: Integer type
------------------------------------------------
Specify the number of double-occupied orbitals for indefinite domainization.

:guilabel:`Frozvir` parameter type: Integer type
------------------------------------------------
Specify the number of virtual orbitals for indefinite domainization.

:guilabel:`Analyze` parameter type: Bool type
------------------------------------------------
Specifies to analyze a user-given fixed-domain orbit, calculating the number of occupied-empty orbital pairs and MOS (Molecular Orbital Spread). To analyze the fixed domain orbits, a file named bdftask.testorb is read from BDF_TMPDIR and the orbits are analyzed. This orbital file is in the same format as SCF's bdftask.scforb, both are text files.

:guilabel:`Iapair` parameter type: floating point
------------------------------------------------
Specify the threshold value for counting the overlap of occupied-empty orbital pairs. By default, only occupied-empty orbital pairs with an absolute overlap value greater than 1.0×10 :sup:`-4`  are counted.

:guilabel:`Directgrid` parameter type: Bool type
--------------------------------------------------
Specifies the calculation of the absolute overlap of the occupied-null orbital pair using the direct numerical integration method.

:guilabel:`Nolmocls` parameter type: Integer type
---------------------------------------------------
Specify the occupancy track of the indefinite SCF.

:guilabel:`Nolmovir` parameter type: Integer type
--------------------------------------------------
Specifies the empty orbit of the indefinite SCF.

:guilabel:`Moprt` parameter type: Integer type
------------------------------------------------
Specify the coefficients for printing the domain-definite molecular orbitals.