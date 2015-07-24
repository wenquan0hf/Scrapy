# 安装指南

## 安装 Scrapy

> 注解
> 
> 请先阅读平台安装指南。

下列的安装步骤假定您已经安装好下列程序:


- [Python](https://www.python.org/)2.7
- PythonPackage:pipandsetuptools。现在 [pip](https://pip.pypa.io/en/latest/installing.html) 依赖 [setuptools](https://pypi.python.org/pypi/setuptools)，如果未安装，则会自动安装setuptools。
- [lxml](http://lxml.de/)。大多数 Linux 发行版自带了 lxml。如果缺失，请查看 `http://lxml.de/installation.html`
- [OpenSSL](https://pypi.python.org/pypi/pyOpenSSL)。除了 Windows(请查看平台安装指南)之外的系统都已经提供。

您可以使用 pip 来安装 Scrapy(推荐使用 pip 来安装 Pythonpackage)。

使用 pip 安装:

```
pip install Scrapy
```

## 平台安装指南

### Windows

- 从 [http://python.org/download/](http://python.org/download/)上安装 Python2.7。

您需要修改 PATH 环境变量，将 Python 的可执行程序及额外的脚本添加到系统路径中。将以下路径添加到 PATH 中:

```
C:\Python27\;C:\Python27\Scripts\;
```

请打开命令行，并且运行以下命令来修改 PATH:

```
c:\python27\python.exec:\python27\tools\scripts\win_add2path.py
```

关闭并重新打开命令行窗口，使之生效。运行接下来的命令来确认其输出所期望的 Python 版本:

```
python--version
```

- 从 [http://sourceforge.net/projects/pywin32/](http://sourceforge.net/projects/pywin32/)安装 pywin32

请确认下载符合您系统的版本(win32 或者 amd64)

- 从 https://pip.pypa.io/en/latest/installing.html 安装 [pip](https://pip.pypa.io/en/latest/installing.html)

打开命令行窗口，确认 `pip` 被正确安装:

```
pip--version
```

- 到目前为止 Python2.7 及 `pip` 已经可以正确运行了。接下来安装 Scrapy:

```
pip install Scrapy
```

### Ubuntu9.10 及以上版本

不要使用 Ubuntu 提供的 python-scrapy，相较于最新版的 Scrapy，该包版本太旧，并且运行速度也较为缓慢。

您可以使用官方提供的 Ubuntu Packages。该包解决了全部依赖问题，并且与最新的 bug 修复保持持续更新。

### Archlinux
您可以依照通用的方式或者从 AUR Scrapy package来安装 Scrapy:

```
yaourt-Sscrapy
```