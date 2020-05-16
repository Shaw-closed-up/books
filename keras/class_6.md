# Keras 入门课6：使用Inception V3模型进行迁移学习

深度学习可以说是一门数据驱动的学科，各种有名的CNN模型，无一不是在大型的数据库上进行的训练。像ImageNet这种规模的数据库，动辄上百万张图片。对于普通的机器学习工作者、学习者来说，面对的任务各不相同，很难拿到如此大规模的数据集。同时也没有谷歌，Facebook那种大公司惊人的算力支持，想从0训练一个深度CNN网络，基本是不可能的。但是好在已经训练好的模型的参数，往往经过简单的调整和训练，就可以很好的迁移到其他不同的数据集上，同时也无需大量的算力支撑，便能在短时间内训练得出满意的效果。这便是迁移学习。究其根本，就是虽然图像的数据集不同，但是底层的特征却是有大部分通用的。  

迁移学习主要分为两种

* 第一种即所谓的transfer learning，迁移训练时，移掉最顶层，比如ImageNet训练任务的顶层就是一个1000输出的全连接层，换上新的顶层，比如输出为10的全连接层，然后训练的时候，只训练最后两层，即原网络的倒数第二层和新换的全连接输出层。可以说transfer learning将底层的网络当做了一个特征提取器来使用。  
* 第二种叫做fine tune，和transfer learning一样，换一个新的顶层，但是这一次在训练的过程中，所有的（或大部分）其它层都会经过训练。也就是底层的权重也会随着训练进行调整。

一个典型的迁移学习过程是这样的。首先通过transfer learning对新的数据集进行训练，训练过一定epoch之后，改用fine tune方法继续训练，同时降低学习率。这样做是因为如果一开始就采用fine tune方法的话，网络还没有适应新的数据，那么在进行参数更新的时候，比较大的梯度可能会导致原本训练的比较好的参数被污染，反而导致效果下降。

本课，我们将尝试使用谷歌提出的Inception V3模型来对一个花朵数据集进行迁移学习的训练。

数据集为17种不同的花朵，每种有80张样本，一共1360张图像，属于典型的小样本集。数据下载地址：http://www.robots.ox.ac.uk/~vgg/data/flowers/17/  
官方没有给出图像对应的label，我写了一段代码，把每张图像加上标签，https://gist.github.com/tsycnh/177bbf7d93adc6207242fd334ce3bb60  
同时，Keras对于数据的格式要求如下：我也写了一个脚本来做转换https://gist.github.com/tsycnh/1b35103adec1ad2be5090c486354859f    
```
data/
    train/
        class1/
            img1
            img2
            ...
        class2/
            img1
            ...
    validation/
        class1/
            img1
            img2
            ...
        class2/
            img1
            ...
    test/
        class1/
            img1
            img2
            ...
        class2/
            img1
            ...
```
这个脚本我将训练集划分为800张，验证集和测试集分别为260张，图片顺序做了随机打乱

如果你懒得自己转换，可以直接下载我已经把处理好的数据：https://download.csdn.net/download/tsyccnh/10641502

请注意，这里的花朵识别仍属于最简单的单分类任务，样张如下

![](./images/flowers.jpg)


```python
from keras.preprocessing.image import ImageDataGenerator
from keras.applications.inception_v3 import InceptionV3,preprocess_input
from keras.layers import GlobalAveragePooling2D,Dense
from keras.models import Model
from keras.utils.vis_utils import plot_model
from keras.optimizers import Adagrad
# 数据准备
train_datagen = ImageDataGenerator(
    preprocessing_function=preprocess_input,# ((x/255)-0.5)*2  归一化到±1之间
    rotation_range=30,
    width_shift_range=0.2,
    height_shift_range=0.2,
    shear_range=0.2,
    zoom_range=0.2,
    horizontal_flip=True,
)
val_datagen = ImageDataGenerator(
    preprocessing_function=preprocess_input,
    rotation_range=30,
    width_shift_range=0.2,
    height_shift_range=0.2,
    shear_range=0.2,
    zoom_range=0.2,
    horizontal_flip=True,
)
```

这里用到的数据集和之前都不同，之前用的是一些公共的、Keras内置的数据集，这次用到的是自己准备的数据集。由于数据的图像大小比较大，不适合一次全部载入到内存中，所以使用了flow_from_directory方法来按批次从硬盘读取图像数据，并实时进行图像增强


```python
train_generator = train_datagen.flow_from_directory(directory='./flowers17/train',
                                  target_size=(299,299),#Inception V3规定大小
                                  batch_size=64)
val_generator = val_datagen.flow_from_directory(directory='./flowers17/validation',
                                target_size=(299,299),
                                batch_size=64)
```

首先我们需要加载骨架模型，这里用的InceptionV3模型，其两个参数比较重要，一个是weights，如果是'imagenet',Keras就会自动下载已经在ImageNet上训练好的参数，如果是None，系统会通过随机的方式初始化参数，目前该参数只有这两个选择。另一个参数是include_top，如果是True，输出是1000个节点的全连接层。如果是False，会去掉顶层，输出一个8 \* 8 \* 2048的张量。 

ps：在keras.applications里还有很多其他的预置模型，比如VGG，ResNet，以及适用于移动端的MobileNet等。大家都可以拿来玩玩。

一般我们做迁移训练，都是要去掉顶层，后面接上各种自定义的其它新层。这已经成为了训练新任务惯用的套路。
输出层先用GlobalAveragePooling2D函数将8 \* 8 \* 2048的输出转换成1 \* 2048的张量。后面接了一个1024个节点的全连接层，最后是一个17个节点的输出层，用softmax激活函数。


```python
# 构建基础模型
base_model = InceptionV3(weights='imagenet',include_top=False)

# 增加新的输出层
x = base_model.output
x = GlobalAveragePooling2D()(x) # GlobalAveragePooling2D 将 MxNxC 的张量转换成 1xC 张量，C是通道数
x = Dense(1024,activation='relu')(x)
predictions = Dense(17,activation='softmax')(x)
model = Model(inputs=base_model.input,outputs=predictions)
# plot_model(model,'tlmodel.png')
```

构建完新模型后需要进行模型的配置。下面的两个函数分别对transfer learning和fine tune两种方法分别进行了配置。每个函数有两个参数，分别是model和base_model。这里可能会有同学有疑问，上面定义了model，这里又将base_model一起做配置，对base_model的更改会对model产生影响么？  
答案是会的。如果你debug追进去看的话，可以看到model的第一层和base_model的第一层是指向同一个内存地址的。这里将base_model作为参数，只是为了方便对骨架模型进行设置。

setup_to_transfer_learning： 这个函数将骨架模型的所有层都设置为不可训练
setup_to_fine_tune：这个函数将骨架模型中的前几层设置为不可训练，后面的所有Inception模块都设置为可训练。  
这里面的GAP_LAYER需要配合打印图和调试的方法确认正确的值，感兴趣具体怎么操作的同学，可以私信我，以后看有没有必要把这个点写成教程。


```python
'''
这里的base_model和model里面的iv3都指向同一个地址
'''
def setup_to_transfer_learning(model,base_model):#base_model
    for layer in base_model.layers:
        layer.trainable = False
    model.compile(optimizer='adam',loss='categorical_crossentropy',metrics=['accuracy'])

def setup_to_fine_tune(model,base_model):
    GAP_LAYER = 17 # max_pooling_2d_2
    for layer in base_model.layers[:GAP_LAYER+1]:
        layer.trainable = False
    for layer in base_model.layers[GAP_LAYER+1:]:
        layer.trainable = True
    model.compile(optimizer=Adagrad(lr=0.0001),loss='categorical_crossentropy',metrics=['accuracy'])
```

下面开始训练，这段代码也演示了如何在全部训练过程中改变模型。


```python
setup_to_transfer_learning(model,base_model)
history_tl = model.fit_generator(generator=train_generator,
                    steps_per_epoch=800,#800
                    epochs=2,#2
                    validation_data=val_generator,
                    validation_steps=12,#12
                    class_weight='auto'
                    )
model.save('./flowers17_iv3_tl.h5')
setup_to_fine_tune(model,base_model)
history_ft = model.fit_generator(generator=train_generator,
                                 steps_per_epoch=800,
                                 epochs=2,
                                 validation_data=val_generator,
                                 validation_steps=1,
                                 class_weight='auto')
model.save('./flowers17_iv3_ft.h5')
```

可以看到经过两个epoch的transfer learning后，验证集准确率达到89.1%。再经过两个epoch的fine tune后验证集准确率达96.88%。可以看到迁移学习的效果还是很好的。

## 总结
1. 学习了两种常用迁移学习方法（tranfer learning,fine tune)及训练技巧
1. 学习了使用自己的数据样本进行训练
1. 学习了加载Keras预置的经典模型
1. 学习了如何在预置模型顶部添加新的层
1. 学习了如何设置层的参数为不可训练



### 参考
> https://deeplearningsandbox.com/how-to-use-transfer-learning-and-fine-tuning-in-keras-and-tensorflow-to-build-an-image-recognition-94b0b02444f2
