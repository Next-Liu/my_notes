DataSet：

```python
from torch.utils.data import Dataset
class Mydata(Dataset): # 继承Dataset, 重写__getitem__(), __len__()的目的是可以使用len()函数
    def __init__(self,root_dir,label_dir):
        self.root_dir = root_dir
        self.label_dir = label_dir
        # 读取文件夹下的所有文件名,保存在列表中
        self.img_path = os.listdir(os.path.join(root_dir,label_dir))
    def __getitem__(self, index):
        img_name = self.img_path[index]# 通过索引获取图片名
        img_item_path = os.path.join(self.root_dir,self.label_dir,img_name)
        img = Image.open(img_item_path)
        label = self.label_dir
        return img,label
    def __len__(self):
        return len(self.img_path)
```

Dataset是一个抽象类，不能直接实例化，需要继承它并重写__getitem__(),__len__()方法,用来获取数据集的大小，以及获取具体的数据



Tensorboard：

```python
from torch.utils.tensorboard import SummaryWriter
writer = SummaryWriter('logs') # 创建一个记录器，logdir为保存数据的路径
# tag 标题
# scalar_value, Y轴的值
# global_step=None, x轴的值
# add_scalar()方法用来记录标量，第一个参数是标签，第二个参数是要记录的值，第三个参数是记录的步数
for i in range(100):
    writer.add_scalar('y=x', i, i)
writer.close()
```

tensorboard是一个可视化工具，可以用来查看模型的结构，训练过程中的loss和acc变化，以及模型参数的分布情况等，使用命令D:\pythonProject\Anaconda\Pytorch>tensorboard --logdir=logs --port=xxxx（**注意要到项目目录下执行，或者使用绝对路径**）

读取图片数据

```python
image_path = 'data/hymenoptera_data/train/bees/17209602_fe5a5a746f.jpg'
img_PIL = Image.open(image_path)
img_array = np.array(img_PIL)
writer.add_image('test', img_array, 2, dataformats='HWC')
#dataformats='HWC'表示数据的维度是(height, width, channels)，1表示步数,接收的数据类型为numpy或者tensor
```



transforms的用法:主要是对图片进行一些变换

常用方法：

**ToTensor**

resize

![fd9a0db21cc58c370c30cdbfe531576](D:\微信文件\WeChat Files\wxid_zbyrxg9ake8b22\FileStorage\Temp\fd9a0db21cc58c370c30cdbfe531576.jpg)

```python
img_path = 'data/hymenoptera_data/train/bees/17209602_fe5a5a746f.jpg'
img = Image.open(img_path)
tensor1 = transforms.ToTensor()
img_tensor = tensor1(img)
#使用OpenCV读取图片，可以直接返回numpy格式
import cv2
cv_img = cv2.imread(img_path)
```

常用transforms

```python
# totensor()方法将PIL图片转换成Tensor
trans_to_tensor = transforms.ToTensor()
img_tensor = trans_to_tensor(img)
print(type(img_tensor))
writer.add_image('Tensor_img', img_tensor)

# normalize()方法将Tensor标准化
trans_norm = transforms.Normalize([0.5, 0.5, 0.5], [0.5, 0.5, 0.5])  # 均值和标准差
img_norm = trans_norm(img_tensor)
# 归一化公式：output[channel] = (input[channel] - mean[channel]) / std[channel]
print(img_tensor[0][0][0])
print(img_norm[0][0][0])
writer.add_image('Normalize_img', img_norm)
writer.close()
```
torchvision下DataSet的使用

`torchvision` 是 PyTorch 的一个计算机视觉库，提供了一系列用于处理图像和视频的工具、数据集加载器、模型结构等功能

1. **数据集加载器：**
   - `torchvision.datasets` 模块提供了多个常用的计算机视觉数据集的加载器，包括 CIFAR-10、CIFAR-100、MNIST、ImageNet 等。
   - 可以通过这些加载器方便地加载和预处理图像数据集。
2. **图像变换和增强：**
   - `torchvision.transforms` 模块包含了各种图像变换和增强操作，如裁剪、旋转、缩放、归一化等。
   - 这些操作可以用于数据增强，提高模型的泛化能力。
3. **模型结构：**
   - `torchvision.models` 模块提供了预训练的深度学习模型，例如 ResNet、VGG、AlexNet 等。
   - 这些模型可以用于图像分类、目标检测等任务。
4. **图像工具：**
   - `torchvision.utils` 模块提供了一些用于图像处理的工具函数，如将图像转为张量、显示图像等。
5. **目标检测和分割：**
   - `torchvision.models.detection` 模块包含了一些用于目标检测的模型，如 Faster R-CNN、SSD 等。
   - `torchvision.models.segmentation` 模块包含了一些用于图像分割的模型，如 DeepLabV3。
6. **视频处理：**
   - `torchvision.io` 模块提供了一些用于读取和写入视频的函数，以及视频转换操作。
7. **功能扩展：**
   - `torchvision.ops` 模块包含了一些计算机视觉相关的运算，如 RoI 池化等。
8. **用于数据加载和处理的通用工具：**
   - `torchvision.utils.data` 模块提供了用于构建数据加载器和数据集的工具。

```python
import torchvision
from torch.utils.tensorboard import SummaryWriter

dataset_transform = torchvision.transforms.Compose([
    torchvision.transforms.ToTensor()
])
# Compose()方法将多个transform组合起来使用
# train代表训练集，download=True表示如果数据集不存在则从网络下载,CIFAR10是一个数据集，包含了10个类别的图片，每个类别有6000张图片，其中50000张用于训练，10000张用于测试
train_dataset = torchvision.datasets.CIFAR10(root='./dataset', transform=dataset_transform, train=True, download=True)
test_dataset = torchvision.datasets.CIFAR10(root='./dataset', transform=dataset_transform, train=False, download=True)
# transform=dataset_transform,表示对数据进行转换，将PIL图片转换成Tensor
# print(test_dataset.classes)  # 打印类别
# img, target = train_dataset[0]
# img.show()  # 展示图片
writter = SummaryWriter('p10')
for i in range(10):
    img, target = test_dataset[i]
    writter.add_image('train_dataset', img, i)
writter.close()
```

DataLoader的使用

加载dataset中的所有数据

```python
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

test_dataset = torchvision.datasets.CIFAR10(root='./dataset', train=False, transform=torchvision.transforms.ToTensor())
test_loader = DataLoader(test_dataset, batch_size=4, shuffle=True, num_workers=0, drop_last=False)
# shuffle=True表示打乱数据，num_workers=2表示使用两个子进程来加载数据,drop_last=False表示如果数据集大小不能被batch_size整除的话，最后剩余多少就不要了
# batch_size=4表示每次读取4张图片
step = 0
writter = SummaryWriter('dataloader')
for data in test_loader:
    imgs, targets = data
    # print(imgs.shape)  # torch.Size([4, 3, 32, 32]) 4表示batch_size,3表示通道数，32表示图片的高和宽
    # print(targets) # tensor([3, 8, 8, 0]) 代表图片的类别
    writter.add_images("test_dataset", imgs, step)
    step += 1
writter.close()
```

python中的__call__方法

`__call__` 方法可以接受任意数量的参数。当你“调用”一个对象时，这些参数将传递给 `__call__` 方法。

```python
pythonCopy codeclass Adder:
    def __init__(self, value=0):
        self.value = value

    def __call__(self, x):
        return self.value + x

add_five = Adder(5)
result = add_five(3)  # 这里我们“调用”了 add_five 对象，结果是 8

```

