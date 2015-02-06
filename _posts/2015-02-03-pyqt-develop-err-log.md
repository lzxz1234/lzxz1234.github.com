---
layout: post
title: PyQt4 开发日志
category : python 
tags : [Python, PyQt, GUI]
---

{% include JB/setup %}

基于 PyQt 的 GUI 开发，界面设计可以通过 Qt Designer 拖拽完成，之后的 ui 文件可以通过 UIC 转换为 python 代码文件，也可以直接加载。

### 加载 ui 文件的两种方式

- 直接加载

{% highlight python %}
uic.loadUi('ui/MainWindow.ui', self)
{% endhighlight %}

- 转换为 python 代码后加载[需要打包 exe 文件时只能通过这种方式]

{% highlight python %}
class MainWindow(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super(MainWindow, self).__init__()
        self.setupUi(self)
{% endhighlight %}

### UIC 转换 ui 文件

{% highlight sh %}
D:\Python27\Lib\site-packages\PyQt4\uic\pyuic.py  $FileDir$\$FileName$ -o $FileDir$\$FileNameWithoutExtension$.py
{% endhighlight %}

### 当前路径获取问题

纯 python 环境下可以通过 `os.base.pathname(__file__)` 获取当前路径，但打包 exe 后该种方式就有问题了，修改后的兼容代码如下：

{% highlight python %}
def cur_file_dir():
    path = sys.path[0]
    if os.path.isdir(path):
        return path
    elif os.path.isfile(path):
        return os.path.dirname(path)
{% endhighlight %}

### 打包代码[基于 py2exe]

{% highlight python %}
#-*-coding:utf-8-*-
from distutils.core import setup
import py2exe
import glob
import sys

sys.argv.append('py2exe')#允许程序通过双击的形式执行

py2exe_options = {
    "includes":["sip"],
    "dll_excludes":["MSVCP90.dll",],
    "compressed":1,#压缩文件
    "optimize":2,#优化级别，默认为0
    "ascii":0,#不自动包含encodings和codecs
    }

setup(
    name='PyQtDemo',
    version='1.0',
    windows=['main.py',],
    zipfile=None,
    options={'py2exe':py2exe_options},
    data_files=[("resources",glob.glob("resources\\*"))]#资源类文件打包到目标路径
)
{% endhighlight %}

### Exe 模式下 Jpg 不显示的问题

Jpg 不显示是由于打包后的程序缺少解码组件，把解码组件打包到目标程序可以解决，转换图片格式也可以。

以下为基于 pillow 的转换代码：

{% highlight python %}
def set_file(self, file_path):
    self.clear()
    self.file_path = file_path
    if file_path and os.path.exists(file_path):
        pix_map = QPixmap()
        if file_path.endswith('.jpg'):
            jpg_data = read(file_path)
            str_io = StringIO.StringIO(jpg_data)
            img = Image.open(str_io)
            img.thumbnail((300,300), Image.ANTIALIAS) 
            png_data = StringIO.StringIO()
            mode = img.mode
            if mode not in ('L', 'RGB'):
                if mode == 'RGBA':
                    alpha = img.split()[3]
                    bg_mask = alpha.point(lambda x: 255 - x)
                    img = img.convert('RGB')
                    img.paste((255, 255, 255), None, bg_mask)
                else:
                    img = img.convert('RGB')
            img.save(png_data, format='png')
            pix_map.loadFromData(QtCore.QByteArray(png_data.getvalue()))
        else:
            pix_map.load(file_path)
        self.addPixmap(pix_map)
{% endhighlight %}

### 启动界面

{% highlight python %}
app = QApplication(sys.argv)

splash_pix = QPixmap("resources/splash.png")
splash = QSplashScreen(splash_pix, QtCore.Qt.WindowStaysOnTopHint)
progressBar = QProgressBar(splash)
progressBar.setAlignment(QtCore.Qt.AlignCenter)
progressBar.setFixedWidth(splash_pix.width())
splash.setMask(splash_pix.mask())
splash.show()

t = QtCore.QElapsedTimer()
t.start()
progressBar.setMaximum(1000)
while (t.elapsed() < 1000):
    progressBar.setValue(t.elapsed())

window = MainWindow()
splash.finish(window)
window.show()
sys.exit(app.exec_())
{% endhighlight %}

### 完整代码
参见 <http://github.com/lzxz1234/Txt2Html>
