---
title: 连连看~辅助程序
date: 2018-11-07 20:56:19
categories: 技术分享
tags: Python
---

# 项目地址
[https://github.com/GitHub-Laziji/lianliankan](https://github.com/GitHub-Laziji/lianliankan)

# 简介
200 行Python 实现的qq连连看 辅助, 用于学习, 请不要拿去伤害玩家们...

# 使用环境
win7
> win10测试了无法使用

# 使用方法
开始游戏后运行就行了, 再次提示, 请在练习模式中使用, 否则可能会被其他玩家举报

# 代码实现
主要思路就是利用`pywin32`获取`连连看游戏句柄`, 获取游戏界面的图片, 对方块进行切割, 对每个方块取几个点的颜色进行比对, 均相同则认为是同一个方块,
然后模拟鼠标去消就行了, 代码的最后一行是每次点击的间隔
```Python
time.sleep(random.randint(0,0)/1000)
```
如果是`0`的话就瞬间全消完了

# 效果图
![1](https://github.com/GitHub-Laziji/lianliankan/raw/master/screenshots/1.png)
![2](https://github.com/GitHub-Laziji/lianliankan/raw/master/screenshots/2.png)

