# 函数

## 回调函数

有些库函数（library function）却要求应用先传给它一个函数，好在合适的时候调用，以完成目标任务。这个被传入的、后又被调用的函数就称为**回调函数**。

```python
import random

Find = {
    'Type': '',
    'Color': '',
    'Size': ''
}   # 定义汇报内容


def callfun(cmd, Find):  # 回调函数的定义，在这里处理各种回调情况
    if cmd == 'Type':
        if Find['Type'] == 'Dog' or Find['Type'] == 'Cat':
            print('A Pet:')
        else:
            print('A Transport:')
    elif cmd == 'Print':
        print(Find)
    else:
        print('error')


def giveinfo(i):    # 该段是填报信息，可忽略
    type0 = ['Dog', 'Cat']
    type1 = ['Car', 'Truck']
    color0 = ['Black', 'White', 'Pink']
    size0 = ['Big', 'Middle', 'Small']
    t0 = i % 2
    if t0 == 0:
        Find['Type'] = type0[i % 2]
    else:
        Find['Type'] = type1[i % 2]
    Find['Color'] = color0[random.randint(0, 2)]
    Find['Size'] = size0[random.randint(0, 2)]


def findobj(num, cmd, callback):     # 发现目标，启动回调函数
    giveinfo(num)   # 门卫填报信息
    callback(cmd, Find)  # 启动回调函数


if __name__ == '__main__':
    cmds = ['Type', 'Print', 'Try']
    for i in range(0, 10):   # 定义十次上报
        print('----------%d-------------' % i)
        findobj(i, cmds[i % 3], callback=callfun)    # 这里注册回调函数（就是告知门卫的过程）
```

回调函数用于解耦



## 函数中的全局变量问题

在一个函数中对全局变量进行修改的时候，到底是否需要使用global进行说明，要看是否对 全局变量的执行指向进行了修改

如果修改了执行，即让全局变量指向了一个新的地方，那么必须使用global

如果，仅仅是修改了 指向的空间中的数据，此时不用必须使用global





