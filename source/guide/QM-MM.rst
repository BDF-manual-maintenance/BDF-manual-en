QM/MM Combination Approach
================================================
The QM/MM combination method generally divides the system into two zones, the QM zone and the MM zone. The total energy of the system is written as:

.. math::

    E_{QM/MM}(\mathbb{S}) = E_{MM}(\mathbb{O})+E_{QM}(\mathbb{I+L})+E_{QM/MM}(\mathbb{I,O}) 

where, S denotes the system, I denotes the QM layer, O denotes the MM layer, and L denotes the linking atom. 
:math:`E_{MM}(\mathbb{O})`  is calculated using molecular mechanics force fields, and  :math:`E_{QM/MM}(\mathbb{I,O})` includes two terms: 

.. math::

    E_{QM/MM}(\mathbb{I,O})=E_{nuc-MM}+V_{elec-MM}

:math:`E_{nuc-MM}+V_{elec-MM}` in the BDF by adding an external point charge to the QM region.

So the total energy of the whole system consists of two parts, :math:`E_{MM}` is calculated using molecular mechanics method, :math:`E_{QM}` 和 :math:`E_{QM/MM}` are calculated using quantum chemical method.
Meanwhile, the interactions between QM and MM regions also include VDW interactions, etc., which will not be discussed here. For the broken bonds in the QM and MM regions, the linked-atom model is generally used to describe them.

The BDF program mainly performs the quantum chemistry calculations, while the rest of the calculations are performed by the pdynamo2.0 program package modified by the group. The rest is done by the pdynamo2.0 package, which has been modified by
the group. See the relevant examples:

.. note::
  
  The installation of the pdynamo program is described in the package. Detailed features of the package can be found in the Help file in the package. This manual only provides instructions and examples for QM/MM calculations using BDF.

  For reference to QM/MM computing environment installation and configuration, see :ref:`QM/MM computing environment configuration<qmmmsetup>`


Input File Preparation
-------------------------------------------------
In general, before QM/MM calculations, molecular dynamics simulations of the target system are required to obtain a suitable initial conformation. Different molecular dynamics software has different output files.
pDynamo-2 currently supports  ``Amber、CHARMM、Gromacs``  and other force fields, and also supports  ``PDB、MOL2、xyz``  and other formats to read in molecular coordinates.

.. note::

    When using PDB, MOL2, or xyz files as input, the pDynamo program only supports the OPLS force field, which is not recommended for small molecules and non-standard amino acid force field parameters are incomplete. It is recommended to prefer the Amber program, which enters the force field parameters through the topology file. If you only do QM calculations, various input methods are available.


In Amber, for example, the structure of interest is extracted from the kinetic simulation trajectory and stored in a ``crd`` file, which together with the corresponding topology file ``.prmtop`` can be used as the starting point for QM/MM
calculations. the Python script is as follows.

.. code-block:: python

  from pBabel import AmberCrdFile_ToCoordinates3, AmberTopologyFile_ToSystem
  # Read input information
  molecule  = AmberTopologyFile_ToSystem(Topfile)
  molecule.coordinates3 = AmberCrdFile_ToCoordinates3(CRDfile)


At this point, the molecule information is stored in the ``molecule`` structure. In the specific QM/MM calculation, energy calculation and geometric configuration optimization of the system are required. Also, the active region can be defined
in the MM region to accelerate the calculation.

Total energy calculation
-------------------------------------------------

Take a 10 Å water box as an example, after the molecular dynamics simulation, the files are extracted as ``wat.prmtop, wat.crd`` , and the full quantum chemical calculation can be performed for the system with the following code:

.. code-block:: python

  import glob, math, os
  from pBabel import AmberCrdFile_ToCoordinates3, AmberTopologyFile_ToSystem
  from pCore import logFile
  from pMolecule mport QCModelBDF,  System
  #  Read the coordinates and topology information of the water box
  molecule = AmberTopologyFile_ToSystem ("wat.prmtop")
  molecule.coordinates3 = AmberCrdFile_ToCoordinates3("wat.crd") 
  # Define the energy calculation mode, which is the density functional calculation of the whole system，GB3LYP:6-31g
  model = QCModelBDF("GB3LYP:6-31g")
  molecule.DefineQCModel(model)
  molecule.Summary()  #Output system calculation setting information
  # Calculate total energy
  energy  = molecule.Energy()

Methods and basis groups can be defined in the  ``QCModelBDF``  class  ``GB3LYP:6-31g`` , with ``:`` partitioning between methods and basis groups. In the above example, it is also possible to select the molecule of interest (e.g., the fifth water molecule)
for the QM/MM calculation, using the QM method for the fifth water molecule and the MM (in this case, the amber force field) for the rest. Since periodic boundary conditions are used in the MD calculation, and the QM/MM method does not support
the use of periodic boundary conditions, an option is added to the script to turn off periodic boundary conditions.

.. code-block:: python

 molecule.DefineSymmetry( crystalClass = None )

The class ``Selection`` is defined in pDynamo and can be used to select specific QM atoms, as described in the usage notes. The script for selecting QM atoms in script is as follows:

.. code-block:: python

 qm_area = Selection.FromIterable(range(12, 15))
 #12. 13 and 14 are atomic list index values (this value = Atomic serial number - 1), which is equal to selecting No. 15 water molecule 
 molecule.DefineQCModel(qcModel, qcSelection = qm_area)

Overall, the script for the combined QM/MM energy calculation is as follows：

.. code-block:: python

  import glob, math, os
  from pBabel import AmberCrdFile_ToCoordinates3, AmberTopologyFile_ToSystem
  from pCore import logFile, Selection
  from pMolecule import NBModelORCA, QCModelBDF,  System
   # . Define the energy models.
  nbModel = NBModelORCA()
  qcModel = QCModelBDF("GB3LYP:6-31g")
  # . Read the data.
  molecule = AmberTopologyFile_ToSystem("wat.prmtop")
  molecule.coordinates3 = AmberCrdFile_ToCoordinates3("wat.crd")
  # .Close symmetry to a system
  molecule.DefineSymmetry(crystalClass = None)   # QM/MM need Close the symmetry.
  # .Selection qm area 
  qm_area = Selection.FromIterable(range (12, 15))  # Select WAT 5 as the QM area.
  # . Define the energy model.
  molecule.DefineQCModel (qcModel, qcSelection = qm_area)
  molecule.DefineNBModel (nbModel)
  molecule.Summary()
  # . Calculate
  energy  = molecule.Energy()

.. note::
  * QM/MM calculation supports two input modes. For simple examples, it can be used as parameter input in ``QCModelBDF`` class.
  
  * relatively complex examples can be input in the form of ``calculation template`` .

Geometry Optimization
-------------------------------------------------
QM/MM geometry optimization is generally not easy to converge and requires a lot of skills in practice. It is common to fix the MM zone and optimize the QM zone, and then fix the QM zone and optimize the MM zone. After several cycles, the QM
and MM regions are optimized at the same time. The convergence of the optimization depends on the choice of the QM zone and the presence of charged atoms at the
QM/MM boundary. In order to speed up the optimization, it is possible to fix the MM region during the calculation and select only a suitable region close to the
QM region as the active region, where the coordinates can change during the optimization. The following arithmetic example for the optimization of geometric configurations is given.


.. code-block:: python

  import glob, math, os.path

  from pBabel import  AmberCrdFile_ToCoordinates3, \
                      AmberTopologyFile_ToSystem , \
                      SystemGeometryTrajectory   , \
                      AmberCrdFile_FromSystem    , \
                      PDBFile_FromSystem         , \
                      XYZFile_FromSystem

  from pCore import Clone, logFile, Selection

  from pMolecule import NBModelORCA, QCModelBDF, System

  from pMoleculeScripts import ConjugateGradientMinimize_SystemGeometry
                             
  # Defines the Opt interface
  def opt_ConjugateGradientMinimize(molecule, selection):
      molecule.DefineFixedAtoms(selection)       # Define fixed atoms
      #Define optimization methods
      ConjugateGradientMinimize_SystemGeometry(
          molecule,
          maximumIterations    =  4,   # Maximum number of optimization steps
          rmsGradientTolerance =  0.1, #Optimize convergence control
          trajectories   = [(trajectory, 1)]
      )   # Defines the frequency at which the track is saved
  # . Define the energy models.
  nbModel = NBModelORCA()
  qcModel = QCModelBDF("GB3LYP:6-31g")
  # . Read the data.
  molecule = AmberTopologyFile_ToSystem ("wat.prmtop")
  molecule.coordinates3 = AmberCrdFile_ToCoordinates3("wat.crd")
  # . Close symmetry to a system
  molecule.DefineSymmetry(crystalClass = None)  # QM/MM need Close the symmetry.
  #. Define Atoms List 
  natoms = len(molecule.atoms)                      # The total number of atoms in the system
  qm_list = range(12, 15)                            # QM region atoms
  activate_list = range(6, 12) + range (24, 27)   # MM region active atom (can be moved during optimization)
  #Defines the MM region atom
  mm_list = range (natoms)
  for i in qm_list:
      mm_list.remove(i)                              # MM deletes the QM atom
  mm_inactivate_list = mm_list[:]
  for i in activate_list :
      mm_inactivate_list.remove(i)                   
  # Enter the QM region atom
  qmmmtest_qc = Selection.FromIterable(qm_list)     # Select WAT 5 as the QM area.
  #  Define each selection zone
  selection_qm_mm_inactivate = Selection.FromIterable(qm_list + mm_inactivate_list)
  selection_mm = Selection.FromIterable(mm_list)
  selection_mm_inactivate = Selection.FromIterable(mm_inactivate_list)
  # . Define the energy model.
  molecule.DefineQCModel(qcModel, qcSelection = qmmmtest_qc)
  molecule.DefineNBModel(nbModel)
  molecule.Summary()
  #Calculate the total energy at the start of the optimization
  eStart = molecule.Energy()
  #Defines the output file
  outlabel = 'opt_watbox_bdf'
  if os.path.exists(outlabel):
      pass
  else:
      os.mkdir (outlabel)
  outlabel = outlabel + '/' + outlabel
  # Define the output trajectory
  trajectory = SystemGeometryTrajectory (outlabel + ".trj" , molecule, mode = "w")
  # Start the first phase of optimization
  # Define two steps to optimization
  iterations = 2
  #  Sequentially fix the QM area and mm area for optimization
  for i in range(iterations):
      opt_ConjugateGradientMinimize(molecule, selection_qm_mm_inactivate) #Fixed QM region optimization
      opt_ConjugateGradientMinimize(molecule, selection_mm)                #Fixed MM region optimization
  # Start the second phase of optimization
  # The QM area and mm area are optimized at the same time
  opt_ConjugateGradientMinimize(molecule, selection_mm_inactivate)
  #Output optimized total energy
  eStop = molecule.Energy()
  #Save optimized coordinates, which can be xyz/crd/pdb, etc。
  XYZFile_FromSystem(outlabel +  ".xyz", molecule)
  AmberCrdFile_FromSystem(outlabel +  ".crd" , molecule)
  PDBFile_FromSystem(outlabel +  ".pdb" , molecule)


QM/MM-TDDFT algorithm example
-------------------------------------------------
After the geometry optimization, the TDDFT calculation can be performed based on the base state calculated by QM/MM. The BDF program interface is designed with a ``calculation template`` function that updates the system coordinates based on the ``.inp`` file given by the user. At the same time, different QM regions can be selected as needed for the geometry optimization and excited state calculation. For example,
in order to consider solvation effects, the first hydrated layer of the molecule of interest can be added to the QM region for QM/MM-TDDFT calculations. Taking the example of the calculation done in the previous section, the calculation can
be continued by adding the following code.


.. code-block:: python

  #Continue with the geometry optimization code from the previous section.
  #Start the TDDFT calculation. Use a template file as input.
  qcModel = QCModelBDF_template(template = 'head_bdf_nosymm.inp') 
  # Adjust the QM region atoms
  tdtest = Selection.FromIterable(qm_list + activate_list)        # Redefine the QM region.
  molecule.DefineQCModel(qcModel, qcSelection = tdtest)
  molecule.DefineNBModel(nbModel)
  molecule.Summary()
  #Energy calculation using the method in the template, (which can be TDDFT)
  energy  = molecule.Energy()

In the above code, the template chosen is the input file of the BDF with the following file contents:

.. code-block:: bdf

 $COMPASS
 Title
  cla_head_bdf
 Basis
  6-31g
 Geometry
  H 100.723 207.273 61.172
  MG   92.917  204.348   68.063
  C   95.652  206.390   67.185
 END geometry
 Extcharge
  point
 nosymm
 $END

 $XUANYUAN
 RSOMEGA
   0.33
 $END
 
 $SCF
 RKS
 DFT
   cam-B3LYP
 $END

 $tddft   #TDDFT calculation control
 iprt
  3
 iroot
  5
 $end





