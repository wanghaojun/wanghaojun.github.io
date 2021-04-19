---
title: tensorflow入门
date: 2019-04-18 15:25:04
tags: [机器学习,深度学习]
---
本文将从tensorflow的计算模型、数据模型和运行模型三个角度介绍tensorflow框架。<!--more-->
# 计算模型——计算图
## 概念
TensorFlow这个名字就说明了两个概念——tensor和flow，tensor是指张量，也就是该框架的数据结构，可以简单的理解为多维数组，flow则体现了框架的计算模型，翻译成中文就是“流”，直观的表达了张量之间通过计算相互转化的过程。TensorFlow是通过计算图的形式来表述计算的编程系统，TensorFlow中的每一个计算都是计算图上的节点，而节点之间的边则描述了计算之间的依赖关系。
## 使用
TensorFlow程序一般分为两个阶段，第一个阶段是定义计算图中的所有计算，第二个阶段为执行阶段，下面代码给出了计算定义阶段的样例。

    import tensorflow as tf
    a = tf.constant([1.0,2.0],name = 'a')
    b = tf.constant([2.0,3.0],name = 'b')
    result = a + b

在TensorFlow程序中，系统会自动维护一个默认的计算图通过函数tf.get_defalut_graph()可以获取当前的默认的计算图。

    #使用a.graph可以获取该张量所属的计算图，因为没有特别指定计算图，
    #所以这个计算图应该等于默认计算图，下面的输出为True
    print(a.graph is tf.get_default_graph())

除了使用默认的计算图之外，TensorFlow还支持使用tf.Graph()函数来生成新的计算图，不同计算图上的张量和运算都不会共享。
    
    import tensorflow as tf
    g1 = tf.Graph()
    with g1.as_default():
        v = tf.get_variable("v", [1], initializer = tf.zeros_initializer()) # 设置初始值为0
    g2 = tf.Graph()
    with g2.as_default():
        v = tf.get_variable("v", [1], initializer = tf.ones_initializer())  # 设置初始值为1
        
    with tf.Session(graph = g1) as sess:
        tf.global_variables_initializer().run()
        with tf.variable_scope("", reuse=True):
            print(sess.run(tf.get_variable("v")))
    with tf.Session(graph = g2) as sess:
        tf.global_variables_initializer().run()
        with tf.variable_scope("", reuse=True):
            print(sess.run(tf.get_variable("v")))
在一个计算图中可以使用集合(collection)来管理不同类型的资源，可以通过tf.add_to_collection()函数来将资源加入一个或者多个集合中，可以使用tf.get_collection()获取一个集合里面的所有资源，这些资源可以是张量，变量或者TensorFlow所需要的队列资源等。

#数据模型——张量
## 概念
从功能的角度看，张量可以被简单的理解为多维数组，其中零阶张量表示标量(slcar)，也就是一个数，第一阶的张量为向量(vector)，也就是一个一维数组，n阶张量可以理解为n维数组。要注意的是张量在TensorFlow中并不是直接采用数组的形式，它只是对TensorFlow计算结果的引用。张量中并没有保存数字，只是保存了如何得到这些数字的计算过程。

    import tensorflow as tf
    a = tf.constant([1.0, 2.0], name="a")
    b = tf.constant([2.0, 3.0], name="b")
    result = a + b
    print (result)
    ''''
    输出：
    Tensor("add:0", shape=(2,), dtype=float32)
    ''''
从上面代码可以看出张量和Numpy中的数组是不同的，TensorFlow的计算结果不是一个具体的数字，而是一个张量的结构，从代码的运行结果可以看出，张量主要保存三个属性：名字(name)，维度(shape)，类型(type).
张量的名字不仅是一个张量的唯一标识符，它同样给出了这个张量是如何计算出来的。张量和计算图上的节点是对应的，这样张量的命名就可以通过"node:src_output"来给出，node为节点名称，src_output表示当前向量来自第几个节点的输出。"add:0"就表示是计算节点"add"输出的第一个结果。
张量的第二个属性是维度，这个属性描述了张量的维度信息，上面代码中的result的维度是(2,)表示是一个一维数组，长度为2.
张量的第三个属性是类型，每个张量会有一个唯一的类型。TensorFlow会对所有参与运算的张量进行检车，发现类型不匹配时会报错。TensorFlow支持14种不同的类型，主要包括了实数(tf.float32,tf.float64)，整数(tf.int8,tf.int16,tf.int32,tf.int64,tf.unit8)，布尔型(tf.bool)和复数(tf.complex64,tf.complex128)
## 使用
张量的使用一般可以归结为两大类，一类是对中间计算结果的引用。可以提高代码的可读性。另一类是当计算图完成构造后，通过张量来获得计算结果，也就是得到真实的数字。
#运行模型——会话
会话(session)拥有并管理TensorFlow程序运行时的所有资源，当计算完成时需要关闭会话来帮助系统回收资源，否则就可能会出现资源泄露的问题，会话的使用方式一般有两种，第一种需要明确调用会话生成和关闭函数：

    sess = tf.Session ()
    sess.run(...)
    sess.close()

这种使用可能会因为程序的异常退出而不执行关闭会话函数，从而导致资源泄露。为了解决资源释放的问题，TensorFlow可以通过Python上下文管理器来使用会话。
    
    with tf.Session() as sess:
    print(sess.run(result))

上文介绍过TensorFlow会自动生成默认的计算图，会话中也有类似的机制，但是需要手动指定默认会话，默认的会话被指定后，可以通过tf.Tensor.eval()函数来计算张量的值。

    sess = tf.Session()
    with sess.as_default():
         print(result.eval())

在交互式的环境下，TensorFlow还提供了一种在交互式环境下直接构建默认会话的函数，就是tf.InteractiveSesssion()，使用这个函数会将自动生成的会话注册为默认会话。

    sess = tf.InteractiveSession ()
    print(result.eval())
    sess.close()

无论用哪种方法都可以通过ConfigProto Protocol Buffer来配置需要生成的会话。

    config=tf.ConfigProto(allow_soft_placement=True, log_device_placement=True)
    sess1 = tf.InteractiveSession(config=config)
    sess2 = tf.Session(config=config)

# 使用TensorFlow实现简单的神经网络
## 神经网络参数和TensorFlow变量

在TensorFlow中，变量(tf.Variable)的作用就是保存和更新神经网络参数的。与其他编程语言类似，也需要指定初始值，初始值一般是随机值。常用的随机生成函数如下：
* tf.random_normal()，正态分布，主要参数为:平均值(mean)，标准差(stddev)，取值类型。
* tf.truncated_normal()，正态分布，但是如果随机值偏离平均值超过2个标准差，那么这个数会被重新随机，主要参数为:平均值(mean)，标准差(stddev)，取值类型。
* tf.random_uniform()，均匀分布，主要参数为：最小，最大值和取值类型。
* tf.random_gamma()，Gamma分布，主要参数：形状参数alpha，尺度参数beta，取值类型。

TensorFlow也支持使用常数来初始化变量，常数生成函数如下：
* tf.zeros()，产生全0数组
* tf.ones()，产生全1数组
* tf.fill(),产生一个全部为给定数字的数组
* tf.constant,产生一个给定值的常量

变量的生成函数tf.Variable()是一个运算，这个运算结果得到的张量就是变量，所以说变量是特殊的张量。tf.Variable()依赖了两种操作，对应计算图两个节点，一种是Assign，这个节点的输入为随机数生成函数的输出，然后将输出赋给了变量，所以这个节点完成了变量的初始化，而另一个节点read，可以将这个变量提供给其他运算。
类似张量，维度和属性也变量最重要的两个属性，变量的类型在构建之后是不可以改变的，但是维度是可以改变的，需要设置参数validate_shape=False。
	
	w1 = tf.Variable(tf.random_normal((2,2),stddev = 1,seed = 1))
	w2 = tf.Variable(tf.random_normal((2,3),stddev = 1,seed = 1))
	tf.assign(w1,w2,validate_shape=False)


## 使用TensorFlow训练神经网络模型
    import tensorflow as tensor
    from numpy.random import RandomState
    #定义batch的大小
    batch_size = 8
    # 定义神经网络参数
    w1 = tf.Variable(tf.random_normal((2,3),stddev = 1,seed = 1))
    w2 = tf.Variable(tf.random_normal((3,1),stddev = 1,seed = 1))
    #定义神经网络的输入
    x = tf.placeholder(tf.float32,shape = (None,2),name = 'input_x')
    y_ = tf.placeholder(tf.float32,shape = (None,1),name = 'input_y')
    #定义向前传播过程
    a = tf.matmul(x,w1)
    y = tf.matmul(a,w2)
    #定义损失函数和反向传播过程
    y = tf.sigmoid(y)
    cross_entropy = -tf.reduce_mean(y_ * tf.log(tf.clip_by_value(y,1e-10,1.0)) + (1-y) * tf.log(tf.clip_by_value(1-y,1e-10,1.0)))
    train_step = tf.train.AdamOptimizer(0.001).minimize(cross_entropy)
    #生成随机数据集
    rdm = RandomState(1)
    dateset_size = 128
    X = rdm.rand(dateset_size,2)
    Y = [[int(x1+x2 < 1)] for (x1,x2) in X]
    #创建会话
    with tf.Session() as sess:
        init_op = tf.global_variables_initializer()
        sess.run(init_op)
        print (sess.run(w1))
        print (sess.run(w2))
        STEP = 5000 #训练的轮数
        for i in range(STEP):
            #选取batch_size大小的数据进行训练
            start = (i * batch_size) % dateset_size
            end = min(start + batch_size,dateset_size)
            #训练神经网络并且更新参数
            sess.run(train_step,feed_dict={x:X[start:end],y_:Y[start:end]})
            #每隔一段时间计算在所有数据上的损失函数并输出
            if (i+1) % 1000 == 0:
                total_cross_entropy = sess.run(cross_entropy,feed_dict={x:X,y_:Y}) 
                print("after %d training steps,cross entropy on all dateset is %g"%(i+1,total_cross_entropy))
        # 打印最终的w1和w2
        print(sess.run(w1))
        print(sess.run(w2))
    ''''
    输出：
    [[-0.8113182   1.4845988   0.06532937]
     [-2.4427042   0.0992484   0.5912243 ]]
    [[-0.8113182 ]
     [ 1.4845988 ]
     [ 0.06532937]]
    after 1000 training steps,cross entropy on all dateset is 0.0685192
    after 2000 training steps,cross entropy on all dateset is 0.0337346
    after 3000 training steps,cross entropy on all dateset is 0.020567
    after 4000 training steps,cross entropy on all dateset is 0.0136916
    after 5000 training steps,cross entropy on all dateset is 0.00963481
    [[-2.5486627  3.0793145  2.895183 ]
     [-4.111281   1.6259116  3.3972812]]
    [[-2.3231003]
     [ 3.3011749]
     [ 2.4632177]]    
    ''''