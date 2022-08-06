TensorFlow软件入门
===================

熟悉TensorFlow基本操作
***********************

2.X版本的TensorFlow软件内置了Keras机器学习接口，极大地降低了程序编写的难度。我们将通过Keras接口调用TensorFlow来建立神经网络。

Google官方提供了丰富的 `TensorFlow快速入门文档 <https://tensorflow.google.cn/tutorials>`_ ，我们主要学习 **面向初学者的快速入门** 和 **Keras机器学习基础知识** 这2个部分。建议跟随教程在我们自己安装的JupyterLab里输入代码，观察代码运行结果，并与教程对照学习。

此外，可以访问 `Keras官方文档 <https://keras.io/zh/>`_ 了解更多Keras的相关知识。因为我们环境中的Keras是TensorFlow内置的而不是单独安装的，因此需要输入下面的代码载入Keras，载入后就可以运行教程中的代码了。

.. code-block::

	from tensorflow import keras as keras