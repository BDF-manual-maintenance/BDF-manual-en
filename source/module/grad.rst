Hartree-Fock Gradient - GRAD Module
================================================
The GRAD module is used to calculate the resolved gradients of HF/MCSCF.
   
**Basic Keywords**   

:guilabel:`Nrootgrad` parameter type: integer
------------------------------------------------
Specifies the gradient of that root for calculating MCSCF.

:guilabel:`Maxiter` Parameter type: integer
------------------------------------------------
Specifies the maximum number of iterations of CPMCHF.

:guilabel:`IntCre` Parameter type: integer
------------------------------------------------
The memory size used to increase the storage of AO points is: intcre*256*1024*1024Bytes.

:guilabel:`Ishell` Parameter type: integer
------------------------------------------------


:guilabel:`Cutcpm` parameter type: floating point
------------------------------------------------
 * Default value：1.D-6

Specifies the convergence threshold for solving the CPMCHF equation.

:guilabel:`Printgrad` parameter type: floating point
------------------------------------------------
 * Selectable values：0、3、>99

Controls gradient printing. 0 is the minimum output; 3 is the contribution of the output single-electron term to the gradient; >99 is for debug mode only.