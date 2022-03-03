
.. _relativity:

Relativistic effects
================================================
The relativistic effect mainly includes two parts: scalar relativistic effect and spin-orbit coupling effect. Relativistic effects are important in the study of organic light-emitting mechanisms, such as inter-system scattering of electronic states due to spin-track coupling, and changes in the spin of transition states and products. The innermost core electrons of the atom are most strongly affected by relativistic effects on the one hand, and insensitive to chemical changes on the other hand, so there are different ways to deal with them, mainly all-electron relativistic and effective core potential (ECP) two types of methods.

1. **All-electron methods** all the inner layer electrons are made variational treatment. The common all-electron relativistic Hamiltonians are: the zero-order regular approximation (ZORA), the second-order Douglas-Kroll-Hess (DKH2), the exact dichotomy (X2C) proposed by Wenjian Liu et al. ZORA and DKH2 have no advantages in terms of accuracy and efficiency and are not recommended.

2. The core electrons of **ECP** heavy atoms are replaced by a pre-fitted effective potential function. When there are more heavy atoms, ECP can significantly reduce the variational degrees of freedom and improve the computational efficiency. The ECP is divided into two categories, pseudopotential (PP) and model core potential (MCP), depending on whether the valence electron wave function has nodes in the core layer or not, where MCP has to combine a large number of Gaussian functions in order to reproduce the valence electron wave function nodes correctly, leading to an insignificant improvement in computational efficiency, and is therefore less used.ECP in BDF programs refers to PP.

The BDF basis set library provides a large number of :ref:` all-electron relativistic basis sets<all-e-bas>` and :ref:`ECP basis sets<ecp-bas>` .


.. warning::

    1. Do not mix X2C Hamiltonian and ECP basis sets
    2. X2C relativistic calculations must use non-shrinking basis sets or specially optimized shrinking basis sets, but the first 18 elements are not mandatory.


Scalar relativistic effects
------------------------------------------------

* **All-electron methods**

The BDF can consider scalar relativistic effects through the spinless X2C Hamiltonian (sf-X2C) and its local approximation variants sf-X2C-AXR, sf-X2C-AU. For example：

.. code-block:: bdf

  $xuanyuan
  heff
   23
  nuclear
    1
  $end

在以上输入中， ``heff`` 调用标量相对论哈密顿，如sf-X2C（3，4，或21），sf-X2C-AXR（原子X矩阵近似，22），
sf-X2C-AU（原子U变换近似，23），其中21，22，23具有解析导数。
对于不涉及5d以上重元素—重元素成键的分子体系，sf-X2C-AU具有最高的效率且不损失精度，是推荐方法。否则用sf-X2C-AXR或sf-X2C。

In the above input, ``heff`` calls scalar relativistic Hamiltonians such as sf-X2C (3,
4, or 21), sf-X2C-AXR (atomic X-matrix approximation, 22), and sf-X2C-AU (atomic
U-transformation approximation, 23), where 21, 22, 23 have analytic derivatives.
For molecular systems not involving heavy element-heavy element bonding above 5d,
sf-X2C-AU has the highest efficiency without loss of precision and is the
recommended method. Otherwise, sf-X2C-AXR or sf-X2C is used.

.. _finite-nuclear:

``nuclear`` 选项设为1表示使用有限核模型，一般不是必须的。但是有些相对论收缩基组考虑了原子核尺寸效应，或者当计算原子核附近的电子性质时，这些情况下要使用有限核模型。

sf-X2C及其局域变体支持的计算类型有：单点能，结构优化，以及部分 :ref:`一阶单电子性质<1e-prop>` 。解析Hessian和二阶单电子性质正在开发中。

* **ECP**

ECP必须与非相对论哈密顿相结合，相对论效应隐含在赝势参数中。
BDF支持的ECP计算类型有：单点能以及部分 :ref:`单电子性质<1e-prop>` 。ECP的梯度和Hessian正在开发中。

旋轨耦合作用
------------------------------------------------
BDF可以通过态相互作用（SI）方法，在TDDFT单点计算中处理不同自旋多重度电子态之间的自旋轨道耦合。需要在 ``xuanyuan`` 模块中通过
``hsoc`` 关键词指定如何计算自旋轨道积分。参见TDDFT部分的示例。

根据采用的哈密顿的不同，自旋轨道耦合也可分为全电子和ECP两类。

* **全电子方法**

虽然双电子自旋轨道积分的贡献小于单电子自旋轨道积分，但是对旋轨耦合作用的影响可能达到1/5~1/3，因此不能忽略。
建议采用单电子自旋轨道积分+单中心近似的分子平均场双电子自旋轨道积分（so1e + SOMF-1c； ``hsoc`` = 2）。
它可以与sf-X2C标量相对论哈密顿结合，对于轻元素体系也可以与非相对论哈密顿结合。
此外，还可以对单电子自旋轨道积分进行屏蔽原子核 :cite:`snso2000,msnso2013` 或有效核电荷 :cite:`zeff1995` 校正，近似考虑双电子自旋轨道积分的影响，但是这对芯层轨道性质可能会造成无法预料的误差。

.. _so1e-zeff:

* **ECP**

包括两种处理方法：

1. 旋轨耦合赝势，需要在标量ECP中加入额外的SO势函数（SOECP；见基组库中的 :ref:`旋轨耦合赝势基组 <soecp-bas>` ），以及
2. 有效核电荷 :cite:`zeff1995,zeff1998` 。

由于双电子自旋轨道相互作用的影响已经包含在SO势的拟合参数或有效核电荷的经验参数中，只要计算单电子自旋轨道积分即可。
在BDF中可以对SOECP描述的原子以及标量ECP或全电子非相对论描述的原子分别使用这两种处理方法，
只需要在 :ref:`xuanyuan<xuanyuan>` 模块中设定 ``hsoc`` 为10。

需要注意的是，有效核电荷支持的元素和基组类型有限。对于全电子基组，仅支持Xe之前的主族元素，且较重的稀有气体元素Ne、Ar、Kr除外。
对于标量ECP基组，虽然支持的元素更多，但是芯电子数必须一致，见下表；

.. table:: 有效核电荷支持的标量ECP基组芯电子数以及原子
    :widths: auto

    +-----------------------------+----------------------------------------+-------+
    | 原子                        | ZA                                     | NCore |
    +=============================+========================================+=======+
    | Li-F                        | 3- 9                                   | 2     |
    +-----------------------------+----------------------------------------+-------+
    | Na-Cl, Sc-Cu, Zn, Ga        | 11-17, 21-29, 30, 31                   | 10    |
    +-----------------------------+----------------------------------------+-------+
    | K -Ca                       | 19-20                                  | 18    |
    +-----------------------------+----------------------------------------+-------+
    | Ge-Br, Y -Ag, Cd, In        | 32-35, 39-47, 48, 49                   | 28    |
    +-----------------------------+----------------------------------------+-------+
    | Rb-Sr                       | 37-38                                  | 36    |
    +-----------------------------+----------------------------------------+-------+
    | Sn-I, La                    | 50-53, 57                              | 46    |
    +-----------------------------+----------------------------------------+-------+
    | Cs-Ba                       | 55-56                                  | 54    |
    +-----------------------------+----------------------------------------+-------+
    | Hf-Au, Hg, Tl               | 72-79, 80, 81                          | 60    |
    +-----------------------------+----------------------------------------+-------+
    | Pb-At                       | 82-85                                  | 78    |
    +-----------------------------+----------------------------------------+-------+

更多细节，如Zeff参数、参考文献等，见soint_util/zefflib.F90。如果有效核电荷方法用于不支持的元素或基组，旋轨耦合计算的结果不可靠。

