输入输出格式
************************************

BDF的输入格式
==========================================================================

BDF输入文件格式有三种，分别为：简洁输入（easy input）；高级输入（advanced input）；混合输入（mixed input）。 **简洁输入** 易用性强，用户无需对计算细节太了解，使用门槛低，面向初级用户。 **高级输入** 提供了对BDF计算强大的控制功能，对每个计算模块都可以精确控制。 **混合输入** 是在BDF简洁输入中，添加部分高级输入模式定义的内容。 BDF的混合输入，是希望在保持BDF简洁输入便利、对初学者友好的基础上，通过增加按照高级输入格式定义的、精确控制BDF计算模块行为的字段，来完成一些高级的计算功能。简而言之，BDF的混合输入是BDF简洁输入与高级输入的组合。对于初学者，只需用BDF简洁输入即可完成大多数计算任务。对于有量子化学理论基础的用户，想深入学习和使用BDF更高级的功能，可以学习采用BDF的高级输入。

.. note::

   * 输入中除 **文件名**、 **shell命令** 及 **环境变量名** 外，其它输入，如BDF的 **模块名** 、 **关键词** 及 **关键词的值** 等都不区分大小写字母。

..


BDF的简洁输入（easy input）
--------------------------------------------------------------------------

以水分子的单点能计算为例来详细描述BDF简洁输入格式：

.. code-block:: bdf

  #!H2O.bdf
  B3lyp/3-21G 

  Geometry  # 输入原子坐标，单位 Angstrom
  O 0.00000    0.00000    0.36827
  H 0.00000   -0.78398   -0.18468
  H 0.00000    0.78398   -0.18468
  End Geometry

BDF简洁输入包含3个输入块：

**第一输入块** 

只有一行，以 ``#!`` 开始，后面是输入脚本的名字，例如 ``#!name.bdf`` , 可以是说明文字。

**第二输入块** 

从第二行开始，到 ``Geometry`` 前一行结束。这一输入块，可以由多行组成，是BDF的命令控制行，用于指定BDF做什么计算任务，可以由多行组成，可以用于指定计算方法，基组、泛函、电荷数及自旋多重度的等一些基本的计算控制参数。命令行内容以空格分开不同的关键词。关键词及其值用等号分开，一个关键词如果没有值，关键词本身即为控制关键词。关键词可以有一个值，也可以有用逗号分开的多个值。
关键词可以有多行，如果一行中出现了 ``#`` ，则 ``#`` 后的行为注释语句。

**第三输入块** 

从 ``Geometry`` 行开始，到 ``End Geometry`` 行结束，输入分子的几何结构，具体格式见分子结构的输入格式说明。

.. tip:: 

  * 本算例的第三行是空行，BDF输入中的空白行,除了在定义分子坐标 ``Geometry ... End geometry`` 之间的，其它的空行都是非必要的，但为了输入的可读性，强烈建议用户用空行分割不同的输入块和不同的模块。

BDF的高级输入（advanced input）
--------------------------------------------------------------------------

BDF高级输入是最初开发BDF时设置的输入模式，特点是 **模块引导计算+模块参数控制**，格式如下：

.. code-block:: bdf

    $bdfmodule1
    # Comment
    Keyword1
      value # inline comment
    Keyword2
      value
      ...
    $end

    %cp $BDFTASK.scforb $BDF_TMPDIR/$BDFTASK.inporb 
    $bdfmodule2
    Keyword1
      value
    Keyword2
      value
      ...
    $end

说明如下：
  - 这个输入包含了两个BDF的计算模块，分别为 ``bdfmodule1`` 和 ``bdfmodule2`` (此处仅为举例，并非真的存在名为 ``bdfmodule1`` 和 ``bdfmodule2`` 的模块)。一个计算任务可能包含多个BDF模块。 
  - **模块引导计算**: 指完成该计算将顺序执行两个计算模块。每个模块的输入从 ``$bdfmodule`` 开始，至随后第一次出现的 ``$end`` 关键词结束, ``$bdfmodule`` 与 ``$end`` 之间是该模块的控制关键词及其值。其中 ``bdfmodule`` 是BDF的计算模块名，如 ``COMPASS``、 ``XUANYUAN`` 、 ``SCF`` 等。
  - **模块参数控制**: 指每个模块通过自有的关键词来控制其计算行为，参数控制输入采用 **关键词+值** 的模式，关键词的值从关键词所在行的下一行开始，根据具体的关键词，可以是单行、也可以是多行。BDF每个计算模块读取并处理自己模块所属的关键词及值。

BDF的高级输入格式需要对量子化学理论有一定了解，也需要知道BDF每个计算模块具体的功能。BDF将不同的计算功能尽可能的独立出来，编译为单独的可执行文件，相当于一个个小工具，然后通过用python语言写的 **bdfdrv.py** 按照计算流程依次调用不同的计算模块来完成复杂的计算任务，各模块之间通过临时文件和环境变量交换数据。

这里，我们以水分子为例来详细描述BDF高级输入格式：

.. code-block:: bdf

  #Example for BDF advanced input
  $compass
  Title
   Water molecule, energy calculation
  Geometry
  O 0.00000    0.00000   0.36827
  H 0.00000   -0.78398   -0.18468
  H 0.00000    0.78398   -0.18468
  End geometry
  Basis # 基组
   3-21G
  Group # C2v点群，可不输入，程序会自动判断，常用于对高阶群指定D2h及其子群计算。
   C(2v)
  $end

  $xuanyuan
  $end

  $scf
  RHF # restricted Hatree-Fock
  $end

  %cp $BDF_WORKDIR/$BDFTASK.scforb $BDF_TMPDIR/$BDFTASK.inporb
  $scf
  RKS # restricted Kohn-Sham
  DFT
   B3lyp     # B3LYP functional， Notice， it is different with B3lyp in Gaussian. 
  Guess 
   Readmo    # Read orbital from inporb as the initial guess orbital
  $end

上面所示的输入文件包含四个计算模块，分别为 **COMPASS**、 **XUANYUAN** 和两个 **SCF** 。 **COMPASS** 用于读入输入分子坐标，基函数等信息，并存储为BDF内部的数据结构。 **COMPASS** 的一个重要任务是对分子点群的处理，包括判断分子对称性，产生对称匹配的轨道（symmetry-adapted orbital）等。 **XUANYUAN** 用于计算单、双电子积分。然后调用两次 **SCF** 模块执行自洽场（self-consistent field）计算，一次为RHF（Restricted Hatree-Fock），另一次为RKS（Restricted Kohn-Sham）。

每个计算模块的输入遵循 **“关键词+值”** 的格式，即给出一个关键词，如 **COMPASS** 中的 ``Group``，紧接着一行为该关键词的值，这里是 ``C(2v)``。有的关键词本身即用于逻辑控制，
如第一个 **SCF** 模块中的 ``RHF``，指定 **SCF** 模块执行 ``RHF`` 计算，这类关键词不需要额外的输入值。而有的关键词的值需要多行输入，具体参见各个模块的关键词说明。

在第一个和第二个 **SCF** 模块之间，有一个 ``%`` 开头的行。这里，我们插入了一条shell命令，执行一个拷贝文件的任务。将第一个 **SCF** 计算产生的放在 **BDF_WORKDIR** 中的 **$BDFTASK.scforb** 文件拷贝到 **BDF_TMPDIR** ，并更名为 **$BDFTASK.inporb** 。
在第二个 **SCF** 模块中，我们指定了关键词 ``guess`` ，值为 ``readmo`` ，即读入分子轨道作为初始猜测。在BDF高级输入中，以 ``%`` 起始的行为shell命令行。输入中以 ``#`` 号开头的行或者行中包含 ``#`` 号，所有的 ``#`` 号后面的内容都是注释语句。

下面的 **BDF模块及计算流程图** 给出了各模块的调用顺序，

.. _BDFpromodules:

.. figure:: images/BDFpromodules.png
   :width: 400
   :align: center

   BDF模块及计算流程图

.. tip::

  - 一个完整的计算任务需调用多个BDF计算模块。高级输入中各模块出现的顺序由 **BDF模块及计算流程图** 给出。一般的计算任务只会涉及上图所示模块中的一小部分，例如大部分计算任务不需要 ``AUTOFRAG`` 模块，第一个计算模块实际上是 ``COMPASS`` ；只有对于 **iOI-SCF** 和 **FLMO** 计算，才应该出现 ``AUTOFRAG`` 模块（并放到 ``COMPASS`` 之前），用以对分子进行自动分片，然后再调用 ``COMPASS`` 等其他计算模块完成工作。
  - 有的计算逻辑较复杂，例如 **分子结构优化** ， 如果利用Kohn-Sham方法优化分子结构， ``COMPASS`` 模块对分子结构，基组等预处理后， ``BDFOPT`` 模块将多次顺序调用 ``XUANYUAN->SCF->RESP`` 三个模块，分别计算单电子积分、自洽场能量及能量对原子核坐标的的梯度优化分子结构。 
  - 实际计算中，BDF的简洁输入文件被翻译为BDF的高级输入格式，存储在 **BDF_ΤΜPDIR** 指定的临时文件夹中的隐藏文件 **.bdfinput** 中。

下面的 **BDF模块及功能表** 给出了BDF的模块名及功能。

.. table:: BDF模块及功能表
    :widths: auto

    ============== =========================================
       模块名          功能 
    ============== =========================================
       AUTOFRAG      分子自动分片，驱动iOI-SCF和FLMO计算
       COMPASS       分子结构、基组及对称性预处理 
       XUANYUAN      原子轨道积分
       BDFOPT        分子几何结构优化
       SCF           Hartree-Fock及Kohn-Sham自洽场 
       TDDFT         含时密度泛函计算
       RESP          Hartree-Fock、Kohn-Sham及TDDFT梯度
       GRAD          Hartree-Fock梯度 
       LOCALMO       分子轨道定域化
       NMR           核磁屏蔽常数计算
       ELECOUP       电子迁移积分，能量迁移积分，定域化激发态
       MP2           Møller-Plesset二级微扰理论 
    ============== =========================================

.. table:: BDF高级输入说明表
    :widths: auto

    ===================== ==============================================================================================================
       输入内容             说明
    ===================== ==============================================================================================================
     $modulename...$end     modulename为BDF计算模块的控制输入,所有的modulename在$BDFHOME/database/program.dat文件中查询
     #号                    #号开始的行或者每行中#号后续的内容均为注释语句
     \*号                   \*号只放于行首，以*号开始的行为注释行
     %号                     %号开始的行，%号后的内容为Shell命令，通常用于处理中间文件
     &database...&end       有些复杂的计算，如FLMO，需要定义分子片段等信息，这通常放于&database与&end之间。请参考 :ref:`test062<test062>`
    ===================== ==============================================================================================================

BDF的混合输入（mixed input）
--------------------------------------------------------------------------

混合输入结合了BDF的简洁输入与高级输入格式，既可享有BDF简洁输入的便利性，又可对BDF的计算模块进行精准的控制，这在执行复杂的计算时非常有用。

BDF混合输入文件的基本结构如下：

.. code-block:: bdf

  #!name.bdf
  方法/泛函/基组 关键词 关键词=选项 关键词=选项1,选项2
  关键词=选项

  Geometry
  分子结构信息
  End Geometry 

  $modulename1
  ...       # 注释语句
  $End

  $modulename2
  ...
  $End


一个混合输入文件可分为4个输入块， **其中前三个输入块是BDF的简洁输入模式的格式** ，第四个输入块， 是 ``End geometry`` 后剩余的内容，与BDF高级输入的格式相同，用于对具体的BDF计算模块的行为进行精确控制，这些参数被加入相应的BDF计算模块中，具有最高的控制优先级。

以水的阳离子为例来详细描述BDF混合输入格式：

.. code-block:: bdf

  #!H2O+.bdf
  B3lyp/3-21G iroot=4 

  Geometry
  O 0.00000    0.00000   0.36827
  H 0.00000   -0.78398   -0.18468
  H 0.00000    0.78398   -0.18468
  End Geometry

  $scf
  Charge # 指定电荷数为+1
   1
  molden # 输出分子轨道为molden格式文件
  $end

上例除了BDF简洁输入的必要内容外，还加入了以 ``$scf`` 开始，到 ``$end`` 结束的行，用以控制 **SCF** 模块。该输入混合了BDF简洁输入和高级输入的内容，在 **SCF** 模块的输入中，加入了关键词 ``charge`` ，设定值为 ``1`` ，用于计算 :math:`\ce{H2O+}` 离子， ``molden`` 关键词控制将SCF收敛后的轨道输出为 **molden** 格式文件，可用于分子结构、轨道、电子密度的可视化，分析波函数，或计算单电子性质。
需指出的是，在混合输入格式的第二行命令行，可以用 ``charge = -1`` 来控制计算 :math:`\ce{H2O+}` 阴离子，但若在后面的scf模块输入中，也使用了 ``charge`` 关键词，则后者具有最高的控制优先级，将覆盖命令行中的输入。换言之，在混合输入格式下，每个BDF计算模块的高级输入关键词具有最高的控制优先级。

分子结构的输入格式
==========================================================================

BDF的分子结构输入从 ``Geometry`` 开始，到 ``End geometry`` 结束，可以按照直角坐标，内坐标，或者指定xyz文件格式的三种方式输入。

.. Warning::
    BDF输入坐标的默认单位为埃（Å），如果需要使用原子单位输入分子结构，需用关键词 ``unit=Bohr`` 来指定。BDF的简洁输入模式下， ``unit=Bohr`` 放在第二行控制行。 如果是高级输入模式，在Compass模块使用关键词 ``unit`` ，并指定值为Bohr。具体见下面的示例。

在简洁输入的控制行指定分子坐标单位，输入的 :math:`\ce{H2}` 分子键长为1.50 Bohr。

.. code-block:: bdf

  #! bdftest.sh
  HF/3-21G unit=Bohr

  Geometry
    H  0.00 0.00 0.00
    H  0.00 0.00 1.50
  End geometry

高级输入模式下，控制分子坐标单位

.. code-block:: bdf

  $compass
  Geometry
    H  0.00 0.00 0.00
    H  0.00 0.00 1.50
  End geometry
  Basis
    3-21G
  Unit
    Bohr
  $end
  
分子结构的直角坐标格式输入
--------------------------------------------------------------------------

.. code-block:: bdf

   Geometry # default coodinate unit is angstrom 
   O  0.00000   0.00000    0.36937
   H  0.00000  -0.78398   -0.18468 
   H  0.00000   0.78398   -0.18468 
   End geometry

.. _Internal-Coord:

分子结构的内坐标格式输入 
--------------------------------------------------------------------------

内坐标采用定义键长、键角、二面角的格式输入，其中键长的单位为埃，键角和二面角的单位为度。输入模式举例如下：

.. code-block:: bdf

   Geometry
   atom1
   atom2 1   R12                  # R12为原子2、1之间键长
   atom3 1   R31  2 A312          # R31为原子3、1之间键长， A312为原子3、1、2定义的键角
   atom4 3   R43  2 A432 1 D4321  # R43为原子4、3之间键长， A432为原子4、3、2定义的键角，D4321为原子4、3、2、1定义的二面角
   atom5 3   R53  4 A534 1 D5341  # R53为原子5、3之间键长， A534为原子5、3、4定义的键角，D5341为原子5、4、3、1定义的二面角 
   ...
   ...
   End Geometry

具体的，对于水分子，内坐标输入如下：

.. code-block:: bdf
 
 Geometry
 O
 H  1   0.9
 H  1   0.9 2 109.0
 End geometry

内坐标输入，利用变量定义内坐标数值如下（ **目前仅简洁输入支持坐标变量！** ）：

.. code-block:: bdf
 
 Geometry
 O
 H  1   R1
 H  1   R1  2  A1        # 定义分子内坐标，坐标值用变量代替

 R1 = 0.9                # 定义坐标变量的值
 A1 = 109.0
 End geometry

.. warning::

    * 内坐标定义注意要保留空白行，内坐标和坐标变量的值之间通过空行分割。

内坐标格式输入，势能面扫描如下（ **目前仅简洁输入支持势能面扫描！** ）：

例1： :math:`\ce{H2O}` 的坐标输入，势能面扫描，键长从0.75埃开始，按照0.05埃的步长，键长由小到大计算20个点。

.. code-block:: bdf
 
 Geometry
 O
 H  1   R1
 H  1   R1  2  109    # 定义分子内坐标，OH键长定义为变量R1

 R1  0.75 0.05 20    # R1的起始值, 扫描步长,扫描点数。 注意保留上一行的空白行
 End geometry

例2： :math:`\ce{H2O}` 势能面扫描的简洁输入，键长从0.75埃开始，按照0.05埃的步长，键长由小到大计算20个点。SCF通过Read获取初始猜测轨道。

.. code-block:: bdf

 #! h2oscan.bdf  
 B3lyp/3-21G Scan Guess=readmo

 Geometry
 O
 H  1   R1
 H  1   R1  2  A1   # 定义分子内坐标，OH键长定义为变量R1, HOH键角为A1

 A1 = 109.0        # 定义键角的值，注意保留上一行空白行

 R1 0.75 0.05 20   # 定义OH键长R1的起始值，扫描步长及扫描点数。
 End geometry


从指定文件中读入分子坐标
--------------------------------------------------------------------------

.. code-block:: bdf
 
 Geometry
 file=filename.xyz    # 需为当前工作下的文件 filename.xyz，只支持xyz格式的输入。
 End geometry


BDF输出文件
==========================================================================

+------------------------------------+------------------------------------------------------------------------------------------+
|            文件扩展名              |     说明                                                                                 |
+====================================+==========================================================================================+
|                  .out              |           主输出文件                                                                     |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .out.tmp          |               结构优化及数值频率任务的副输出文件（包含能量、梯度等计算步骤的输出）       |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .pes1             | 结构优化及数值频率任务各步的分子结构（埃）、能量（Hartree）及梯度（Hartree/Bohr）        |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .egrad1           |           结构优化及数值频率任务最后一步的能量（Hartree）及梯度（Hartree/Bohr）          |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .hess             |                                    Hessian矩阵（Hartree/Bohr^2）                         |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .unimovib.input   |                                    UniMoVib输入文件，可用于热化学分析                    |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .nac              |                      非绝热耦合矢量（Hartree/Bohr）                                      |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .chkfil           |            临时文件                                                                      |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .datapunch        |            临时文件                                                                      |
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .optgeom          |  标准取向下的分子坐标（Bohr）。其中对于结构优化任务，为结构优化最后一步的分子坐标        |
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .finaldens        |           最后一步SCF迭代的密度矩阵                                                      | 
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .finalfock        |           最后一步SCF迭代的Fock矩阵                                                      | 
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .scforb           |           最后一步SCF迭代的分子轨道                                                      |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .global.scforb    |           FLMO/iOI计算最后一步SCF迭代的分子轨道                                          |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .fragment*.*      |           FLMO/iOI计算的子体系计算相关输出文件                                           |  
+------------------------------------+------------------------------------------------------------------------------------------+
|                  .ioienlarge.out   |           iOI计算第1步及之后的宏迭代的子体系组成信息                                     |  
+------------------------------------+------------------------------------------------------------------------------------------+



某些计算任务可能会产生以上所未列举的其他输出文件，这些文件一般为临时文件。


量子化学常用单位及换算
==========================================================================

量子化学程序大部分内部运算使用原子单位制（atomic unit, a.u.）。这使得各种计算公式中不需要涉及单位转换，既使得代码简洁，也避免额外的运算和精度损失。量化程序输出中间数据时一般也用原子单位制，但输出有化学意义的数据时大多还是会转换成常用的单位。

 * 能量 1 a.u. = 1 Hartree
 * 质量 1 a.u. = 1 m :sub:`e` （电子质量）
 * 长度 1 a.u. = 1 Bohr = 0.52917720859 Å
 * 电量 1 a.u. = 1 e = 1.6022×10 :sup:`-19` C
 * 电子密度 1 a.u. = 1e/Bohr :sup:`3`
 * 偶极矩 1 a.u. = 1 e · Bohr = 0.97174×10 :sup:`22` V/m :sup:`2` = 2.5417462 Debye
 * 静电势 1 a.u. = 1 Hartree/e
 * 电场 1 a.u. = 1 Hartree/(Bohr · e) = 51421 V/Å

能量单位换算
----------------------------------------------

+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+
| 1 unit =          | Hartree             | kJ·mol :sup:`-1`    | kcal·mol :sup:`-1`  |      eV             |  cm :sup:`-1`     |
+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+
| Hartree           |   1                 |    2625.50          |     627.51          |    27.212           | 2.1947×10 :sup:`5`|
+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+
| kJ·mol :sup:`-1`  | 3.8088×10 :sup:`-4` |     1               |     0.23901         | 1.0364×10 :sup:`-2` |   83.593          |
+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+
| kcal·mol :sup:`-1`| 1.5936×10 :sup:`-3` |     4.184           |     1               | 4.3363×10 :sup:`-2` |   349.75          |
+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+
|    eV             | 3.6749×10 :sup:`-2` |     96.485          |     23.061          |       1             |   8065.5          |
+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+
|    cm :sup:`-1`   | 4.5563×10 :sup:`-6` | 1.1963×10 :sup:`-2` | 2.8591×10 :sup:`-3` | 1.2398×10 :sup:`-4` |       1           |
+-------------------+---------------------+---------------------+---------------------+---------------------+-------------------+

长度单位换算
----------------------------------------------

+-------------------+---------------------+---------------------+---------------------+
| 1 unit =          | Bohr                |     Å               |         nm          |
+-------------------+---------------------+---------------------+---------------------+
| Bohr              |   1                 |    0.52917720859    |     0.052917720859  |
+-------------------+---------------------+---------------------+---------------------+
| Å                 | 1.88972613          |     1               |     0.1             |
+-------------------+---------------------+---------------------+---------------------+
|     nm            | 0.188972613         |     10              |     1               |
+-------------------+---------------------+---------------------+---------------------+
