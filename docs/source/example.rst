代码样例参考
=============

以下是相关代码样例，供大家编写代码时参考。
	
建立层流火焰速度数据库代码
***************************

编写循环，利用Cantera软件对不同压力、温度、当量比和掺氢比工况下混合燃料的层流火焰速度进行计算，并将结果保存下来，形成层流火焰速度数据库。

.. code-block::
	
	import cantera as ct
	import numpy as np
	import time
	
	# computational time log
	start_time = time.perf_counter()
	
	width = 0.03  # m
	loglevel = 0  # amount of diagnostic output (0 to 8)
	air = "O2:0.21,N2:0.79"
	
	# Solution object used to compute mixture properties, set to the state of the
	# upstream fuel-air mixture
	gas = ct.Solution('A1NTC.cti')
	
	flame_speed = []
	for iphi in range(8, 13):
		phi = iphi/10
		#print(phi)
		for ip in range(1,5):
			p = ip*ct.one_atm  # pressure [Pa]
			#print(p)
			for iTin in range(300, 550, 50):
				Tin = iTin
				#print(Tin)
				for iH2 in range(0, 4):
					H2_ratio = iH2/10
					#print(H2_ratio)
			
					fuel = "POSF10264:"+str(1.0-H2_ratio)+",H2:"+str(H2_ratio)  # 燃料组分，POSF10264是航空煤油
					gas.TP = Tin, p
					gas.set_equivalence_ratio(phi=phi, fuel=fuel, oxidizer=air)
					
					f = ct.FreeFlame(gas, width=width)
					f.set_refine_criteria(ratio=3, slope=0.06, curve=0.12)
					
					f.transport_model = 'Multi'
					
					solve_time = time.perf_counter()
					f.solve(loglevel=loglevel, auto=True)
					
					print('phi=%.1f, p=%datm, T=%d, H2=%.1f' % (phi, ip, Tin, H2_ratio))
					print('    multicomponent flamespeed = {0:7f} m/s'.format(f.velocity[0]))
					print('    solve time=%.1fmin, total time=%.1fmin' % ((time.perf_counter()-solve_time)/60.0, (time.perf_counter()-start_time)/60.0))
					flame_speed.append(f.velocity[0])

	# 保存层流火焰速度数据库
	np.save('flame_speed.npy',flame_speed)

训练神经网络模型代码
*********************

载入前面建立好的层流火焰速度数据库，利用TensorFlow软件训练人工神经网络模型来预测层流火焰速度。

.. code-block::

	import tensorflow as tf
	import numpy as np
	
	# 载入层流火焰速度数据库
	flame_speed = np.load('flame_speed.npy')
	
	# 构建训练数据集
	train_input = np.zeros([len(flame_speed),4])
	
	i = 0
	for iphi in range(8, 13):
		phi = iphi/10
		#print(phi)
		for ip in range(1,5):
			p = ip  # pressure [atm]
			#print(p)
			for iTin in range(300, 550, 50):
				Tin = iTin
				#print(Tin)
				for iH2 in range(0, 4):
					H2_ratio = iH2/10
					
					train_input[i] = [p,Tin,phi,H2_ratio]
					i = i+1
					
	train_label = flame_speed.copy()
	
	# 建立人工神经网络模型
	model = tf.keras.models.Sequential([
	tf.keras.layers.InputLayer(input_shape=[4]),
	tf.keras.layers.Dense(128, activation='relu'),
	tf.keras.layers.Dense(64, activation='relu'),
	tf.keras.layers.Dense(1),
	])
	model.compile(optimizer=tf.keras.optimizers.Adam(learning_rate=0.0001),
				loss='mean_squared_error',
				metrics=['mean_absolute_error', 'mean_squared_error'])
	
	# 训练模型
	train_fit = model.fit(train_input, train_label, batch_size=32, epochs=200)
	
	# 保存训练过程中损失值的变化
	loss = train_fit.history['loss']
	np.savetxt("loss.csv", loss, delimiter=",")
	
	# 保存训练好的模型
	model.save('ANN_flame_speed.h5')
	
.. Note::

	可以把训练过程中损失值的变化用Excel软件绘制成曲线图。神经网络的训练过程就是通过优化网络参数让损失值不断降低的过程。
	
检验神经网络模型预测效果代码
*****************************

随机生成原数据库中不存在的新工况，利用训练好的神经网络模型预测航空煤油掺混氢气的层流火焰速度，并与Cantera的计算结果进行对比，观察神经网络模型对于未知新工况的预测准确度。

.. code-block::

	import tensorflow as tf
	import cantera as ct
	import numpy as np
	
	# 随机选择工况
	rand = np.random.rand() # 生成0-1之间的随机数
	p0 = 1+(4-1)*rand #压力
	
	rand = np.random.rand() # 生成0-1之间的随机数
	Tin = 300+(500-300)*rand #温度
	
	rand = np.random.rand() # 生成0-1之间的随机数
	phi = 0.8+(1.2-0.8)*rand #当量比
	
	rand = np.random.rand() # 生成0-1之间的随机数
	H2_ratio = 0+(0.3-0)*rand # 掺氢比
	
	# 采用神经网络预测层流火焰速度
	model = tf.keras.models.load_model('ANN_flame_speed.h5')
	test_input = np.zeros([1,4])
	test_input[0] = [p0,Tin,phi,H2_ratio]
	ANN_pred = model.predict(test_input)
	print('ANN predicted flamespeed = {0:7f} m/s'.format(ANN_pred[0,0]))
	
	# 采用Cantera计算层流火焰速度
	# Simulation parameters
	p = p0*ct.one_atm  # pressure [Pa]
	
	fuel = "POSF10264:"+str(1.0-H2_ratio)+",H2:"+str(H2_ratio)  # 燃料组分，POSF10264是航空煤油
	air = "O2:0.21,N2:0.79"
	
	width = 0.03  # m
	loglevel = 0  # amount of diagnostic output (0 to 8)
	
	# Solution object used to compute mixture properties, set to the state of the
	# upstream fuel-air mixture
	gas = ct.Solution('A1NTC.cti')
	gas.TP = Tin, p
	gas.set_equivalence_ratio(phi=phi, fuel=fuel, oxidizer=air)
	
	# Set up flame object
	f = ct.FreeFlame(gas, width=width)
	f.set_refine_criteria(ratio=3, slope=0.06, curve=0.12)
	
	# Solve with multi-component transport properties
	f.transport_model = 'Multi'
	f.solve(loglevel=loglevel, auto=True)
	
	print('multicomponent flamespeed = {0:7f} m/s'.format(f.velocity[0]))
	
	print('Conditions: P = %.2f, T = %.2f, Phi = %.2f, H2 = %.2f' % (p0, Tin, phi, H2_ratio))
	print('ANN predicted flamespeed = {0:7f} m/s'.format(ANN_pred[0,0]))
	print('Cantera predicted flamespeed = {0:7f} m/s'.format(f.velocity[0]))
	
.. Note::

	重复运行这段代码，即可随机生成不同的新工况。大家可以重复5次，在论文中列出神经网络模型和Cantera预测的层流火焰速度进行对比，分析神经网络模型的预测准确度。