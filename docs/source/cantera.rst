利用Cantera软件计算层流火焰速度
===============================

我们打开JupyterLab网页，点击Notebook下方的Python 3按钮，进入程序编写界面。

.. Note::
	#. 如果JupyterLab已经在运行，那么只需要在浏览器地址栏输入 `localhost:8888 <http://localhost:8888>`_ 即可重新打开网页;
	#. 如果JupyterLab尚未运行，那么在开始菜单找到Anaconda Navigator打开，在右边找到JupyterLab，点击 **Launch** 按钮加载JupyterLab工具。

.. Warning::
	JupyterLab默认新建一个名为Untitled.ipynb文件，我们可以在左侧边栏右击或者按F2将这个文件重命名，比如命名为test.ipynb
	
熟悉Cantera基本操作
********************

我们打开 `官方文档 <https://cantera.org/tutorials/python-tutorial.html>`_ 学习Cantera的基本操作，主要看一下 **Getting Started** 和 **Setting the State** 这2个部分。

学习层流火焰速度的计算范例
***************************

层流火焰速度是燃料燃烧的一种重要基础特性。Cantera官方提供了一个氢气层流火焰速度的 `计算范例代码 <https://cantera.org/examples/python/onedim/adiabatic_flame.py.html>`_ 。其原理是利用Cantera数值求解氢气的一维层流预混火焰，当火焰位置稳定后未燃反应物气体的流速等于层流火焰的传播速度。