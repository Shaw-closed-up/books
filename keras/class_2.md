# Keras入门课2：使用CNN识别mnist手写数字


```python
import keras
from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras import backend as K
```


```python
(x_train,y_train),(x_test,y_test) = mnist.load_data() # out: np.ndarray
print(x_train.shape,y_train.shape)
print(x_test.shape,y_test.shape)
```

↓可视化一些图片


```python
import matplotlib.pyplot as plt
im = plt.imshow(x_train[0],cmap='gray')
plt.show()
im2 = plt.imshow(x_train[1],cmap='gray')
plt.show()
```


```python
print(K.image_data_format())
```

这里用卷积神经网络来对图像做特征处理，一般来说，输入到网络的图像格式有以下两种：
1. channels_first (batch_size,channels,width,height)
1. channels_last  (batch_size,width,height,channels)

这里channels指的是通道数，灰度图是单通道channels=1，彩色图是三通道channels=3，需要注意的是，即使图像是单通道的，输入数据的维度依然是4维。反观我们的mnist图像数据，只有三维，所以我们要手动把channels这个维度加上。由于Keras使用不同后端的时候，数据格式不一样，所以要分情况进行维度增加

值得注意的是，reshape函数第一个参数为-1，意思为保持当前维度不变


```python

if K.image_data_format()=='channels_first':
    x_train = x_train.reshape(-1,1,28,28)
    x_test = x_test.reshape(-1,1,28,28)
    input_shape = (1,28,28)
else:
    x_train = x_train.reshape(-1,28,28,1)
    x_test = x_test.reshape(-1,28,28,1)
    input_shape = (28,28,1)
```


```python
print(x_train.shape,x_test.shape)
```

↓数据归一化


```python
x_train = x_train/255
x_test = x_test/255
```


```python
y_train = keras.utils.to_categorical(y_train,10)
y_test = keras.utils.to_categorical(y_test,10)
```

↓构建网络模型


```python
model = Sequential()
model.add(Conv2D(filters = 32,kernel_size=(3,3),
                 activation='relu',input_shape = input_shape))
model.add(Conv2D(64,(3,3),activation='relu'))
model.add(MaxPooling2D(pool_size=(2,2)))
model.add(Dropout(0.25))#25%的参数会被舍弃
model.add(Flatten())
model.add(Dense(128,activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(10,activation='softmax'))
```


```python
model.summary()
```


```python
model.compile(loss = keras.losses.categorical_crossentropy,
             optimizer = keras.optimizers.Adadelta(),
             metrics=['accuracy'])
```


```python
model.fit(x_train,y_train,batch_size=64,epochs=2
          ,verbose=1,validation_data=(x_test,y_test))
```


```python
score = model.evaluate(x_test, y_test, verbose=0)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```

## 总结

1. 学习了如何根据不同的模型数据要求，给原始数据图像增加维度
2. 学习了Conv2D卷积层和MaxPooling2D池化层的使用

本文代码地址：https://github.com/tsycnh/Keras-Tutorials/blob/master/class_2.ipynb

参考：
> https://github.com/keras-team/keras/tree/master/examples


```python

```
