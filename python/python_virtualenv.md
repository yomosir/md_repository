# python virtualenv

## advantages

python项目的环境容易互相干扰，导致项目运行失败。有虚拟环境可以让每个项目拥有自己独立的python环境，保证在依赖这方面项目可以保证稳定运行。

## examples

1. 安装虚拟环境包

```
	pip install virtualenv
```

2. 在项目中创建虚拟环境

```
	# 进入项目目录,安装virtualenv默认python版本创建虚拟环境
	virtualenv --no-site-packages venv
```

3. 进入并激活虚拟环境

```
	source venv/bin/active
```

## requirement.txt
搭配虚拟环境使用，可以先将项目中所安装的python包及版本给写入到一个txt文件中，然后到项目部署的地方在进行安装，保证环境的一致性

1. 生成txt文件
```
	pip freeze > requirement.txt
```

2. 安装txt文件中的模块
```
	pip install -r requirement.txt
```