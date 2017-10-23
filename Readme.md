### 关于安装
本篇讲述的是关于A卡rmbp的caffe安装与编译中一些问题的解决。

### Caffe版本
我download下的是目前最新的caffe1.0。
关于download caffe1.0的包，直接git clone下来
> git clone https://github.com/BVLC/caffe.git

### 配合食用的工具
brew基本都用。没有下载homebrew的盆友请如下操作安装homebrew及依赖项
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install wget
```
之后的caffe编译过程建议选择cmake，cmake使用在后面会说。我们会选用brew进行安装
### 需要的环境
#### 1.python与c++环境准备
首先是关于python的问题。之前本地上安装的2.7和3.6，但是在安装过程中改动Makefile的过程中始终会导致numpy连接到System下的pythonframework里。由于这个问题有两个解决方法，后面会详细说道。如果不想释放权限，可以选择安装anaconda，对于之后Makefile.config中的配置比较友好。如果想关掉Rootless对Python.framework中的文件进行编辑，选择进入恢复模式csrutil disabled即可。然而这种头很铁的行为我不推荐。之后会以anaconda配置python的方式便于进行caffe python接口的编译。
BAIR的install文档关于libstdc++ installation的部分没什么需要多说的，基本上都不会因为标准库困扰。
#### 2.关于anaconda
不能翻的盆友可以直接去tsinghua找一份anaconda的安装包，为了更好的支持caffe1.0，建议选择python2.7版本进行下载。这给一份下载链接：
>https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda2-5.0.0-MacOSX-x86_64.pkg

之后懒人包安装。对环境变量进行配置。视个人使用在~/.bashrc或~/.zshrc(宇宙第一fish fish~/.config/fish/config.fish去set)中添加
>export PATH=~/anaconda/bin:$PATH

之后关于anaconda的使用问题直接直接-h即可，我们直接使用anaconda建立并激活python2.7环境
```
conda create --name python27 python=2.7
activate python27
source activate python27
```
其他包的管理可以选择pip或者conda install进行。
#### 3.其他依赖项安装
根据BAIR的官方文档进行安装。
```
brew install -vd snappy leveldb gflags glog szip lmdb
# need the homebrew science source for OpenCV and hdf5
brew tap homebrew/science
brew install hdf5 opencv
```
如果之前安装过snappy等库，卸了更新，写个脚本sh一下，依赖项也这部分可以参见官方文档
```
for x in snappy leveldb gflags glog szip hdf5 lmdb homebrew/science/opencv;
do
    brew uninstall $x;
    brew install --fresh -vd $x;
done
brew install --build-from-source --with-python -vd protobuf
brew install --build-from-source -vd boost boost-python
```

### caffe编译安装
caffe down下来以后，开始caffe的安装和编译。进入的caffe根目录，以我本地为例如下：

![](http://oy6b059ev.bkt.clouddn.com/caffe%E5%AE%89%E8%A3%851.png)

之后准备进行caffe的编译。在这里先要根据你的需要动一下Makefile.config。为了开启python接口和选择只用CPU模式，我们需要对原生makefile的一些部分进行修改
首先是CPU_ONLY。将line8的cpu_only=1 uncomment掉，改成只用CPU的模式

![](http://oy6b059ev.bkt.clouddn.com/cpu_only1.png)

之后要进行python和matlab接口安装的配置。如果想要开启matlab接口，将下图的matlab部分uncomment掉并根据你的matlab路径进行设置。我开了python接口的配置。python接口给了你两个选择。没有安装anaconda的话你可以选择将你本地的python2.7路径和numpy/core作为python_include的值，但是每次这样做最后都会发现无法import numpy.core.multiarray，print出numpy路径后会发现这里没有链到lib/python而是链到了/System/Library/Framework中的Python.Frameworks中的numpy库，其中numpy的版本不符合caffe1.0的要求。因此要不改掉rootless，用更新的numpy替代。这里有链接关于此问题的解决：
>https://stackoverflow.com/questions/20518632/importerror-numpy-core-multiarray-failed-to-import

如果不想这么改动，请启用anaconda进行python环境设置。之前有讲到anaconda的环境设置，之后需要在anaconda设置的环境中装好需要的库即可。此时python路径如下图中的anaconda几行配置完成

![](http://oy6b059ev.bkt.clouddn.com/pylib.png)

原生的caffe本身不带build和cmake这些目录。我们进入到caffe根目录后，先建立一个build文件夹并在其下进行编译
```
mkdir build
cmake ..
```
cmake过后会贴出一些信息，展示caffe的安装依赖项是否已经安装齐和版本是否满足要求，以及你需要安装的接口、以及模式的设置，请仔细阅读，如有缺失项请回到caffe根目录下make clean清除并重新按照上面的一些步骤与说明安装依赖项，并重复上述make过程。

![](http://oy6b059ev.bkt.clouddn.com/cmake1.png)

之后需要对两个文件改动,一个是cmakecache.txt,另一个是caffeconfig.cmake。其中关于CPU_ONLY的设置还是off，所以需要改动。

![](http://oy6b059ev.bkt.clouddn.com/buildcpu_only.png)
![](http://oy6b059ev.bkt.clouddn.com/set.png)

看完配置信息，如果觉得没问题，继续make
>make all

![](http://oy6b059ev.bkt.clouddn.com/makeall.png)
make all之后会提示你pycaffe的信息，之后
>make pycaffe

![](http://oy6b059ev.bkt.clouddn.com/pycaffe.png)

okay，整个过程配置完毕，最后结果如下：

![](http://oy6b059ev.bkt.clouddn.com/caffedone.png)

之后去examples跑一下caffe的例子就可以了。可以看一下转化lmdb等等数据格式的一些预备知识在进行食用。
