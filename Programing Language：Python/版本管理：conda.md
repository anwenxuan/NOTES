# 一、conda

在Linux上使用软件时，有的软件需要的版本不一样，比如Python就有2和3这两个版本的差别，在进行爬虫时，就需要python2的版本，这时可以使用conda创建一个新的环境
 conda create -n py2.7 python=2.7

激活环境
 source activate py2.7

退出环境
 source deactivate

删除环境
 conda remove -n py2.7 --all

查询有哪些环境
 conda info -e