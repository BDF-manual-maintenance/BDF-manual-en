
.. _Point-Group:

对称性与分子点群
================================================

BDF支持在计算中考虑分子点群对称性。除某些计算任务（如开壳层TDDFT、TDDFT/SOC等）仅支持 :math:`\rm D_{2h}` 及其子群（即 :math:`\rm C_1, C_i, C_s, C_2, D_2, C_{2h}, C_{2v}, D_{2h}` ，一般统称为阿贝尔群）以外，大部分计算任务支持任何的实表示点群（所有的阿贝尔群，以及 :math:`\rm C_{nv}, D_{n}, D_{nh}, D_{nd}, T_d, O, O_h, I, I_h` ；其中特殊点群 :math:`\rm C_{\infty v}, D_{\infty h}` 虽然名义上支持，但是是分别当作 :math:`\rm C_{20v}` 和 :math:`\rm D_{20h}` 来处理的，而单个原子按 :math:`\rm O_{h}` 群处理），但不支持复表示点群（ :math:`\rm C_n, C_{nh} (n \ge 3); S_{2n} (n \ge 2); T, T_h` ）。程序可以自动根据用户在COMPASS模块输入的分子坐标来判断分子所属的点群，并在分子属于复表示点群时自动改用合适的子群。确定分子所属点群以后，程序即可产生该点群的群操作算符、特征标表、不可约表示等信息，以备后续计算使用。以氨分子为例：

.. code-block:: bdf

    #! NH3.sh
    HF/cc-pVDZ 

    geometry
     N                 -0.00000000   -0.00000000   -0.10000001
     H                  0.00000000   -0.94280900    0.23333324
     H                 -0.81649655    0.47140450    0.23333324
     H                  0.81649655    0.47140450    0.23333324
    end geometry

    $compass
    Title
      NH3
    thresh
      medium
    $end

对应的高级输入模式，**COMPASS** 中的内容为

.. code-block:: bdf

  $COMPASS
  Title
   NH3
  Basis
   cc-pvdz
  Geometry
   N                 -0.00000000   -0.00000000   -0.10000001
   H                  0.00000000   -0.94280900    0.23333324
   H                 -0.81649655    0.47140450    0.23333324
   H                  0.81649655    0.47140450    0.23333324
  End geometry
  thresh
   medium 
  $END

注意因为初始结构不严格满足 :math:`\rm C_{3v}` 对称性，这里用 ``thresh medium`` 选择较松的判断对称性的阈值（默认为 ``tight`` ，也可选择更松的 ``loose`` ）。由输出文件可以看到，程序自动识别出该分子属于 :math:`\rm C_{3v}` 点群：

.. code-block:: 

  gsym: C03V, noper=    6
   Exiting zgeomsort....
   Representation generated
    Point group name C(3V)                        6
    User set point group as C(3V)
    Largest Abelian Subgroup C(S)                         2

注意点群名称的下标需要用括号括起来，诸如 :math:`\rm C_{\infty v}, D_{\infty h}` 群需要写作C(LIN)、D(LIN)。接下来打印不可约表示信息、CG系数表等。在COMPASS部分输出的最后，程序给出该点群下不可约表示的列表，以及属于每个不可约表示的轨道的数目：

.. code-block:: 

  |--------------------------------------------------|
            Symmetry adapted orbital

    Total number of basis functions:      29      29

    Number of irreps:   3
    Irrep :     A1        A2        E1
    Norb  :     10         1        18
  |--------------------------------------------------|
  
不可约表示的排列顺序
---------------------------------------------

很多时候，用户需要在输入文件中指定诸如每个不可约表示的轨道占据数（在SCF模块的输入中指定），以及每个不可约表示下计算多少个激发态（在TDDFT模块的输入中指定）等信息，而这些信息一般是以数组的形式提供的，例如

.. code-block:: bdf

  $TDDFT
  Nroot
   3 1 2
  $END

表示第一个不可约表示计算3个激发态，第二个不可约表示计算1个激发态，第三个不可约表示计算2个激发态（详见本手册的 :ref:`TDDFT章节<TD>`）。这就势必要求用户在撰写输入文件时已知道各个不可约表示在BDF程序内部的排列顺序。以下给出BDF支持的所有点群下各个不可约表示的排列顺序：

.. table:: 不同点群下各个不可约表示的排列顺序
   :widths: 30 70

   ==================== ======================================================================================================
   C(1)                 A
   C(i)                 Ag, Au
   C(s)                 A', A"
   C(2)                 A, B
   C(2v)                A1, A2, B1, B2
   C(2h)                Ag, Bg, Au, Bu
   D(2)                 A, B1, B3, B2
   D(2h)                Ag, B1g, B3g, B2g, Au, B1u, B3u, B2u
   C(nv) (n=2k+1, k>=1) A1, A2, E1, ..., Ek
   C(nv) (n=2k+2, k>=1) A1, A2, B1, B2, E1, ..., Ek
   D(n)  (n=2k+1, k>=1) A1, A2, E1, ..., Ek
   D(n)  (n=2k+2, k>=1) A1, A2, B1, B2, E1, ..., Ek
   D(nh) (n=2k+1, k>=1) A1', A2', E1', ..., Ek', A1", A2", E1", ..., Ek", 
   D(nh) (n=2k+2, k>=1) A1g, A2g, B1g, B2g, E1g, ..., Ekg, A1u, A2u, B1u, B2u, E1u, ..., Eku
   D(nd) (n=2k+1, k>=1) A1g, A2g, E1g, ..., Ekg, A1u, A2u, E1u, ..., Eku
   D(nd) (n=2k+2, k>=1) A1', A2', B1', B2', E1', ..., Ek', A1", A2", B1", B2", E1", ..., Ek"
   T(d)                 A1, A2, E, T1, T2
   O                    A1, A2, E, T1, T2
   O(h)                 A1g, A2g, Eg, T1g, T2g, A1u, A2u, Eu, T1u, T2u
   I                    A, T1, T2, F, H
   I(h)                 Ag, T1g, T2g, Fg, Hg, Au, T1u, T2u, Fu, Hu
   ==================== ======================================================================================================

用户也可强制程序在分子所属点群的某个子群下计算，方法是在COMPASS模块的输入里使用group关键词，如：

.. code-block:: bdf

  #! N2.sh
  HF/def2-TZVP group=D(2h) 

  geometry
    N  0.00 0.00 0.00
    N  0.00 0.00 1.10
  end geometry

或者

.. code-block:: bdf

  $COMPASS
  Title
   N2
  Basis
   def2-TZVP
  Geometry
   N 0.00 0.00 0.00
   N 0.00 0.00 1.10
  End geometry
  Group
   D(2h)
  $END

即强制程序在 :math:`\rm D_{2h}` 点群下计算 :math:`\rm N_2` 分子，尽管 :math:`\rm N_2` 分子实际上属于 :math:`\rm D_{\infty h}` 点群。注意程序会自动检查用户输入的点群是否是分子实际所属点群的子群；如否，则程序报错退出。

标准取向 (standard orientation)
---------------------------------------------

为了计算以及结果分析方便起见，程序在确定计算所用点群以后，会将分子旋转到标准取向，以使得分子的对称轴尽量和坐标轴重合，对称面尽量和坐标轴垂直。这样的好处在于可以让计算涉及的很多量精确等于0（如某些分子轨道系数，梯度的某些分量等），方便分析计算结果。

BDF按照以下规则确定分子的标准取向：

1. 将分子的所有原子坐标按核电荷取加权平均，得到分子的核电荷中心，然后平移分子使得核电荷中心位于坐标系原点；
2. 如果分子有对称轴，将分子的最高阶对称轴（主轴）旋转至z轴方向；
3. 如果分子有 :math:`\sigma_v` 对称面，将其中一个 :math:`\sigma_v` 对称面旋转至xz平面方向，过程中保证主轴方向不变；
4. 如果分子除主轴外还有其他的二重轴或四重轴，将其中一根轴（如果存在四重轴，则选择任意一根四重轴，否则选择任意一根二重轴）旋转至x轴方向，过程中保证主轴方向不变；
5. 如果因为分子的对称性太低，以上各条件不能唯一确定分子的取向，则旋转分子使得分子的惯性轴（即转动惯量的本征矢）和各坐标轴方向一致。

对于某些特殊情形，以上规则仍无法唯一确定分子的取向。例如属于 :math:`\rm C_{2v}` 点群的分子，因有两个 :math:`\sigma_v` 对称面，在上述规则的第3步时任一个对称面均有可能被旋转到xz方向。在BDF里，如水分子等平面结构的 :math:`\rm C_{2v}` 分子会被旋转到xz平面：

.. code:: bdf

  |-----------------------------------------------------------------------------------|

   Atom   Cartcoord(Bohr)               Charge Basis Auxbas Uatom Nstab Alink  Mass
    O   0.000000  -0.000000   0.219474   8.00   1     0     0     0   E     15.9949
    H  -1.538455   0.000000  -0.877896   1.00   2     0     0     0   E      1.0073
    H   1.538455  -0.000000  -0.877896   1.00   2     0     0     0   E      1.0073

  |------------------------------------------------------------------------------------|

相比之下其他量化程序则可能选择将分子旋转至yz平面。由此会带来另一个问题：根据习惯约定， :math:`\rm C_{2v}` 点群下 :math:`\mathbf{x}` 算符属于B1不可约表示， :math:`\mathbf{y}` 算符属于B2不可约表示，因此如果某量化程序选择将分子转至yz平面，则其B1、B2不可约表示的定义和BDF是相反的，即该程序的B1表示对应于BDF的B2表示，该程序的B2表示对应于BDF的B1表示。而如果该 :math:`\rm C_{2v}` 点群的分子不是平面结构（如环氧乙烷），则更加难以预测BDF中分子的标准取向是否和其他量化软件一致。因此如果用户希望计算 :math:`\rm C_{2v}` 点群的分子，并与其他量化程序的结果相比较（或者试图重复文献用其他量化程序计算出来的结果），则用户必须确认该量化程序的B1、B2表示是如何和BDF对应的。