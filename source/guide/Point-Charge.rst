
Point charge model
================================================
The BDF supports the calculation of atomic charges in the MM region as point charge input. The point charge is given as input to the ``$BDFTASK.extcharge`` file
with the same name as the calculation task as input, in the following format:

.. code-block:: bdf

  $COMPASS
  Title
  water molecule in backgroud of exteral charges
  Basis
    6-31g
  Geometry
  O   0.000000   0.000000   0.106830
  H   0.000000   0.785178  -0.427319
  H   0.000000  -0.785178  -0.427319
  End Geometry
  Extcharge  #Indicates that an input point charge is required
    point      #Indicates that the input charge type is a point charge
  $END
  
  $XUANYUAN
  $END

  $SCF
  RHF
  $END

The point charge input file (file name h2o.extcharge) is as follows：

.. code-block:: bdf

    External charge, Point charge   #The first line is the title and description line
    6                               #The number of point charges to enter 
    C1     -0.732879     0.000000     5.000000     0.114039 
    C2      0.366440     0.000000     5.780843    -0.456155 
    C3      0.366440     0.000000     4.219157    -0.456155
    C4     -0.732879     0.000000     10.00000     0.114039 
    C5      0.366440     0.000000     10.78084    -0.456155 
    C6      0.366440     0.000000     9.219157    -0.456155

The default input format for point charges is: atomic label, charge, and coordinates (x\  y\  z); the coordinates are in angstroms by default. Coordinate input
units can also be be Bohr, and the input format is as follows


.. code-block:: bdf

    External charge, Point charge   # title line
    6    Bohr                       # Unit: Bohr  
    C1     -0.732879     0.000000     5.000000     0.114039 
    #     ... # 



