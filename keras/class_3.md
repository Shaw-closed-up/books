# Keras入门课3：使用CNN识别cifar10数据集

cifar10是一个日常物品的数据集，一共有10类，属于是比较小的数据集。这次用一个4个卷积层加2个全连接层的典型CNN网络来进行分类


```python
import keras
from keras.datasets import cifar10
from keras.preprocessing.image import ImageDataGenerator
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten
from keras.layers import Conv2D, MaxPooling2D
```

↓首先载入cifar10数据集，和mnist数据集的载入方法一致，本地没有数据的话会先下载


```python
(x_train,y_train),(x_test,y_test) = cifar10.load_data()
```

cifar10数据集图像大小是32*32的3通道彩图，训练集5万张，测试集1万张。和之前的mnist数据集不同，由于是彩色的，所以样本直接就是4维的。


```python
print(x_train.shape,y_train.shape)
print(x_test.shape,y_test.shape)
```


```python
import matplotlib.pyplot as plt
plt.imshow(x_train[0])
plt.show()
plt.imshow(x_train[1])
plt.show()
```

可以看到数据读入没有问题，第一张是蛤蟆，第二张是一个卡车。

↓规范化数据


```python
x_train = x_train/255
x_test = x_test/255
y_train = keras.utils.to_categorical(y_train,10)
y_test = keras.utils.to_categorical(y_test,10)
```

↓构建模型。之前构建模型都是先生成一个model，然后使用add方法来一层一层的加，现在用另一种更方便的方法。直接在Sequential初始化的时候按数组一个一个写进去就可以了。


```python
model = Sequential([
    Conv2D(32,(3,3),padding='same',input_shape=(32,32,3),activation='relu'),
    Conv2D(32,(3,3),activation='relu'),
    MaxPooling2D(pool_size=(2,2)),
    Dropout(0.25),
    
    Conv2D(64,(3,3),padding='same',activation='relu'),
    Conv2D(64,(3,3),activation='relu'),
    MaxPooling2D(pool_size=(2,2)),
    Dropout(0.25),
    
    Flatten(),
    Dense(512,activation='relu'),
    Dropout(0.5),
    Dense(10,activation='softmax')    
])
```


```python
model.summary()
```

↓指定优化函数的参数


```python
opt = keras.optimizers.rmsprop(lr=0.0001,decay=1e-6)
```


```python
model.compile(loss='categorical_crossentropy',
             optimizer=opt,
             metrics=['accuracy'])
```

### 至此直接调用fit方法就可以进行训练了。但是为了模型更快的收敛以及更好的泛化性能，往往我们会对图像做一些变换，比如缩放、平移、旋转等等。下面我们要用keras自带的图像增强来对图像做一些变换

↓这里生成了一个数据增强器，包含了范围20°内的随机旋转，±15%的缩放以及随机的水平翻转。可调的参数还有很多，具体的可以查看文档。


```python
datagen = ImageDataGenerator(
    rotation_range = 20,
    zoom_range = 0.15,
    horizontal_flip = True,
)
```


```python
# datagen.fit(x_train) 只有使用featurewise_center，featurewise_std_normalization或zca_whitening时需要此函数
```

↓通过ImageDataGenerator生成的数据需要使用model的fit_generator方法来进行训练，其中的workers参数表示多线程运算。

datagen的flow方法可以按批次的生成训练所需数据，注意这里生成的数据都是经过了数据增强的，并且是实时的。


```python
model.fit_generator(datagen.flow(x_train,y_train,batch_size=64),steps_per_epoch = 1000,epochs = 2,
                    validation_data=(x_test,y_test),workers=4,verbose=1)
```

↓保存模型，包括了模型的结构以及参数。后缀用h5


```python
model.save('cifar10_trained_model.h5')
```


```python
scores = model.evaluate(x_test,y_test,verbose=1)
print('Test loss:',scores[0])
print('Test accuracy:',scores[1])
```

### 总结

1. 学习了一种新的使用Sequential()构建模型的方法，更加的简单快捷
1. 学习了使用Keras内置的ImageDataGenerator来做数据增强的方法
1. 调用model的fit_generator来进行针对增强数据的训练
1. 学习了如何保存模型

本文代码链接：https://github.com/tsycnh/Keras-Tutorials/blob/master/class_3.ipynb

参考
> https://github.com/keras-team/keras/blob/master/examples
> https://keras-cn.readthedocs.io/en/latest/preprocessing/image/
