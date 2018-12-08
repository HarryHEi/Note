---
title: Packaging Python Projects
date: 2018-12-08 10:39:00
tags: [python]
---

# 打包Python项目
[Packaging Python Projects](https://packaging.python.org/tutorials/packaging-projects/)

## 项目结构
打包的项目`project`结构如下:
```
project/
├── project
│   ├── __init__.py
│   └── ***.py
├── LICENSE
├── README.md
├── setup.py
└── venv
```

需要在`project/project/__init__.py`写上名称，用来确认是否安装成功
```
name = 'project'
```

## setup.py
`setup.py`是`setuptools`的build脚本，用来管理包的依赖文件。
```
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="example_pkg",
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)
```

## LICENSE
[LICENSE](https://choosealicense.com/)

## 安装依赖
需要确认`setuptools`和`wheel`是最新版本
```
python -m pip install --user --upgrade setuptools wheel
```

## 生成wheel
使用命令生成`dist`文件夹
```
python setup.py sdist bdist_wheel
```

如下:
```
project/dist/
├── project-0.0.1-py3-none-any.whl
└── project-0.0.1.tar.gz
```

如果增加版本号再生成wheel，新增的版本也会保存到dist文件夹
```
project/dist/
├── project-0.0.1-py3-none-any.whl
├── project-0.0.1.tar.gz
├── project-0.0.2-py3-none-any.whl
└── project-0.0.2.tar.gz
```

## 上传
安装依赖
```
python -m pip install --user --upgrade twine
```

先上传到TestPyPi测试
```
twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```

上传到PyPi
```
twine upload dist/*
```

当需要更新版本时，只需要上传更新的版本，比如
```
twine upload dist/project-0.0.2*
```

不允许上传同名称同版本的包，即时当一个包删除时，也不允许上传同名称包。
