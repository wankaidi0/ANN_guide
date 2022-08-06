训练人工神经网络预测层流火焰速度
=================================

使用TensorFlow软件构建人工神经网络（Artificial Neural Network, ANN）模型，预测航空煤油掺混氢气的层流火焰速度。
	
建立人工神经网络模型
*********************

使用TensorFlow软件可以非常快速方便地构建我们需要的神经网络模型。这里我们不需要深入理解TensorFlow和神经网络的原理，只需要掌握使用方法即可。我们的神经网络模型需要4个输入，即 **压力、温度、当量比和掺氢比** 。根据这4个输入数据，神经网络模型需要预测该工况下航空煤油掺混氢气的层流火焰速度，因此神经网络模型的输出只有1个，即 **层流火焰速度** 。

打开JupyterLab网页，在之前新建的case_conda文件夹下新建ANN.ipynb文件，输入下列代码建立人工神经网络：

.. code-block::
	
	import tensorflow as tf
	
	model = tf.keras.models.Sequential([
		tf.keras.layers.InputLayer(input_shape=[4]),
		tf.keras.layers.Dense(128, activation='relu'),
		tf.keras.layers.Dense(64, activation='relu'),
		tf.keras.layers.Dense(1),
		])
		
	model.compile(optimizer=tf.keras.optimizers.Adam(learning_rate=0.001),
					loss='mean_squared_error',
					metrics=['mean_absolute_error', 'mean_squared_error'])

其中，tf.keras.models.Sequential表示我们建立的是一个序列模型，也就是从输入到输出是一个简单的顺序结构，中间没有分支、合并等复杂网络结构。

tf.keras.layers.InputLayer是输入层，设定了输入数据有4个。最后一层tf.keras.layers.Dense是输出层，设定了输出数据有1个。中间两层tf.keras.layers.Dense是中间层，也叫隐含层，每层的神经元节点数量设定为128和64个；activation激活函数采用了ReLU函数。

model.compile中设定了如下参数：optimizer优化器选用了Adam方法，learning_rate学习速率设定为0.001，loss损失函数采用了均方误差，metris评价指标设置为平均绝对误差和均方误差2个。


构建训练数据集
***************

一组训练数据为压力、温度、当量比和掺氢比这4个输入数据和对应的层流火焰速度。训练数据集则是若干组训练数据的集合。一般来说，训练数据集越丰富越好，这样训练出来的神经网络模型预测效果更准确。

作为示例，这里我们构建一个简单的训练数据集，包含5组训练数据：

.. code-block::

	import numpy as np
	
	train_input = np.zeros([5,4])
	train_label = np.zeros([5])
	
	train_input[0] = [1, 300, 0.8, 0]
	train_label[0] = 0.278139
	
	train_input[1] = [3, 400, 0.9, 0.2]
	train_label[1] = 0.401458
	
	train_input[2] = [4, 500, 1.0, 0.3]
	train_label[2] = 0.605466
	
	train_input[3] = [1, 350, 1.1, 0.3]
	train_label[3] = 0.461102
	
	train_input[4] = [2, 450, 1.2, 0.1]
	train_label[4] = 0.537888

其中，train_input中每一行的数据为压力、温度、当量比和掺氢比，train_label中的数据为Cantera计算得到的该工况下的层流火焰速度。


训练神经网络模型
*****************

只需要一行代码即可进行神经网络模型的训练：

.. code-block::

	model.fit(train_input, train_label, batch_size=2, epochs=50)
	
其中，batch_size是设定每一批次训练几组数据，我们设定为2组，epochs是指对整个训练数据集循环多少次，我们设定为50次。训练过程中，程序会不断输出loss和error的情况。我们可以看到随着训练的进行，loss和error是在不断下降的。

.. code-block::

	Epoch 1/50
	3/3 [==============================] - 1s 4ms/step - loss: 621.9543 - mean_absolute_error: 23.9366 - mean_squared_error: 621.9543
	Epoch 2/50
	3/3 [==============================] - 0s 3ms/step - loss: 126.1330 - mean_absolute_error: 8.8534 - mean_squared_error: 126.1330
	Epoch 3/50
	3/3 [==============================] - 0s 3ms/step - loss: 273.9980 - mean_absolute_error: 16.1513 - mean_squared_error: 273.9980
	Epoch 4/50
	3/3 [==============================] - 0s 12ms/step - loss: 39.1323 - mean_absolute_error: 5.6953 - mean_squared_error: 39.1323
	Epoch 5/50
	3/3 [==============================] - 0s 5ms/step - loss: 24.0898 - mean_absolute_error: 4.6771 - mean_squared_error: 24.0898
	Epoch 6/50
	3/3 [==============================] - 0s 4ms/step - loss: 94.6176 - mean_absolute_error: 9.5424 - mean_squared_error: 94.6176
	Epoch 7/50
	3/3 [==============================] - 0s 5ms/step - loss: 40.2508 - mean_absolute_error: 6.1616 - mean_squared_error: 40.2508
	Epoch 8/50
	3/3 [==============================] - 0s 10ms/step - loss: 2.5859 - mean_absolute_error: 1.5413 - mean_squared_error: 2.5859
	Epoch 9/50
	3/3 [==============================] - 0s 7ms/step - loss: 29.8354 - mean_absolute_error: 5.3781 - mean_squared_error: 29.8354
	Epoch 10/50
	3/3 [==============================] - 0s 6ms/step - loss: 29.4403 - mean_absolute_error: 5.2932 - mean_squared_error: 29.4403
	Epoch 11/50
	3/3 [==============================] - 0s 11ms/step - loss: 3.5210 - mean_absolute_error: 1.5579 - mean_squared_error: 3.5210
	Epoch 12/50
	3/3 [==============================] - 0s 3ms/step - loss: 9.3138 - mean_absolute_error: 2.9066 - mean_squared_error: 9.3138
	Epoch 13/50
	3/3 [==============================] - 0s 5ms/step - loss: 13.2016 - mean_absolute_error: 3.4854 - mean_squared_error: 13.2016
	Epoch 14/50
	3/3 [==============================] - 0s 5ms/step - loss: 1.2594 - mean_absolute_error: 0.9644 - mean_squared_error: 1.2594
	Epoch 15/50
	3/3 [==============================] - 0s 6ms/step - loss: 2.5840 - mean_absolute_error: 1.5430 - mean_squared_error: 2.5840
	Epoch 16/50
	3/3 [==============================] - 0s 3ms/step - loss: 4.2440 - mean_absolute_error: 2.0444 - mean_squared_error: 4.2440
	Epoch 17/50
	3/3 [==============================] - 0s 7ms/step - loss: 0.5990 - mean_absolute_error: 0.6711 - mean_squared_error: 0.5990
	Epoch 18/50
	3/3 [==============================] - 0s 6ms/step - loss: 1.8664 - mean_absolute_error: 1.3161 - mean_squared_error: 1.8664
	Epoch 19/50
	3/3 [==============================] - 0s 5ms/step - loss: 2.0592 - mean_absolute_error: 1.3977 - mean_squared_error: 2.0592
	Epoch 20/50
	3/3 [==============================] - 0s 10ms/step - loss: 0.1225 - mean_absolute_error: 0.2670 - mean_squared_error: 0.1225
	Epoch 21/50
	3/3 [==============================] - 0s 14ms/step - loss: 0.8520 - mean_absolute_error: 0.9161 - mean_squared_error: 0.8520
	Epoch 22/50
	3/3 [==============================] - 0s 11ms/step - loss: 0.5746 - mean_absolute_error: 0.7230 - mean_squared_error: 0.5746
	Epoch 23/50
	3/3 [==============================] - 0s 3ms/step - loss: 0.0464 - mean_absolute_error: 0.1780 - mean_squared_error: 0.0464
	Epoch 24/50
	3/3 [==============================] - 0s 11ms/step - loss: 0.4360 - mean_absolute_error: 0.6446 - mean_squared_error: 0.4360
	Epoch 25/50
	3/3 [==============================] - 0s 4ms/step - loss: 0.1062 - mean_absolute_error: 0.3005 - mean_squared_error: 0.1062
	Epoch 26/50
	3/3 [==============================] - 0s 7ms/step - loss: 0.1453 - mean_absolute_error: 0.3543 - mean_squared_error: 0.1453
	Epoch 27/50
	3/3 [==============================] - 0s 6ms/step - loss: 0.2184 - mean_absolute_error: 0.4553 - mean_squared_error: 0.2184
	Epoch 28/50
	3/3 [==============================] - 0s 9ms/step - loss: 0.0197 - mean_absolute_error: 0.1023 - mean_squared_error: 0.0197
	Epoch 29/50
	3/3 [==============================] - 0s 9ms/step - loss: 0.0967 - mean_absolute_error: 0.2926 - mean_squared_error: 0.0967
	Epoch 30/50
	3/3 [==============================] - 0s 4ms/step - loss: 0.0278 - mean_absolute_error: 0.1361 - mean_squared_error: 0.0278
	Epoch 31/50
	3/3 [==============================] - 0s 10ms/step - loss: 0.0247 - mean_absolute_error: 0.1405 - mean_squared_error: 0.0247
	Epoch 32/50
	3/3 [==============================] - 0s 4ms/step - loss: 0.0439 - mean_absolute_error: 0.1981 - mean_squared_error: 0.0439
	Epoch 33/50
	3/3 [==============================] - 0s 4ms/step - loss: 0.0038 - mean_absolute_error: 0.0547 - mean_squared_error: 0.0038
	Epoch 34/50
	3/3 [==============================] - 0s 7ms/step - loss: 0.0342 - mean_absolute_error: 0.1581 - mean_squared_error: 0.0342
	Epoch 35/50
	3/3 [==============================] - 0s 6ms/step - loss: 0.0127 - mean_absolute_error: 0.0750 - mean_squared_error: 0.0127
	Epoch 36/50
	3/3 [==============================] - 0s 7ms/step - loss: 0.0226 - mean_absolute_error: 0.1278 - mean_squared_error: 0.0226
	Epoch 37/50
	3/3 [==============================] - 0s 15ms/step - loss: 0.0152 - mean_absolute_error: 0.1044 - mean_squared_error: 0.0152
	Epoch 38/50
	3/3 [==============================] - 0s 4ms/step - loss: 0.0076 - mean_absolute_error: 0.0823 - mean_squared_error: 0.0076
	Epoch 39/50
	3/3 [==============================] - 0s 18ms/step - loss: 0.0227 - mean_absolute_error: 0.1338 - mean_squared_error: 0.0227
	Epoch 40/50
	3/3 [==============================] - 0s 7ms/step - loss: 0.0082 - mean_absolute_error: 0.0756 - mean_squared_error: 0.0082
	Epoch 41/50
	3/3 [==============================] - 0s 17ms/step - loss: 0.0105 - mean_absolute_error: 0.0792 - mean_squared_error: 0.0105   
	Epoch 42/50
	3/3 [==============================] - 0s 3ms/step - loss: 0.0102 - mean_absolute_error: 0.0896 - mean_squared_error: 0.0102
	Epoch 43/50
	3/3 [==============================] - 0s 6ms/step - loss: 0.0089 - mean_absolute_error: 0.0793 - mean_squared_error: 0.0089
	Epoch 44/50
	3/3 [==============================] - 0s 4ms/step - loss: 0.0081 - mean_absolute_error: 0.0752 - mean_squared_error: 0.0081
	Epoch 45/50
	3/3 [==============================] - 0s 6ms/step - loss: 0.0062 - mean_absolute_error: 0.0694 - mean_squared_error: 0.0062
	Epoch 46/50
	3/3 [==============================] - 0s 12ms/step - loss: 0.0065 - mean_absolute_error: 0.0705 - mean_squared_error: 0.0065
	Epoch 47/50
	3/3 [==============================] - 0s 5ms/step - loss: 0.0061 - mean_absolute_error: 0.0682 - mean_squared_error: 0.0061
	Epoch 48/50
	3/3 [==============================] - 0s 7ms/step - loss: 0.0067 - mean_absolute_error: 0.0723 - mean_squared_error: 0.0067
	Epoch 49/50
	3/3 [==============================] - 0s 3ms/step - loss: 0.0066 - mean_absolute_error: 0.0724 - mean_squared_error: 0.0066
	Epoch 50/50
	3/3 [==============================] - 0s 3ms/step - loss: 0.0063 - mean_absolute_error: 0.0690 - mean_squared_error: 0.0063

训练完成后，我们可以调用model.predict函数进行层流火焰速度的预测：

.. code-block::

	model.predict(train_input)
	
程序输出结果如下：
	
.. code-block::
	
	array([[0.29831234],
           [0.51501596],
           [0.6595675 ],
           [0.3535203 ],
           [0.49524155]], dtype=float32)
		   
可以看到，神经网络模型预测的结果与train_label中的目标值有一定的偏差，说明训练的步数还不够，可以把epochs值设定更大再进行训练。

.. Warning::
	大家按照本示例的方法，把前面构建的层流火焰速度的完整数据库用于训练。神经网络模型的预测准确度与训练数据集的大小和范围密切相关， **数据量大、范围广的训练数据集** 能够显著提升神经网络模型的预测效果。
	
.. Note::

	训练好的神经网络模型可以用于预测数据库中不存在的新工况的层流火焰速度，请大家自行选择新的 **压力、温度、当量比和掺氢比** 输入到神经网络模型中，预测航空煤油掺混氢气的层流火焰速度，并与Cantera的计算结果进行对比，观察神经网络模型对于未知新工况的预测准确度。