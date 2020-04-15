---
layout:     post
title:      "「游戏中的数学」 2 四元数与欧拉角"
subtitle:   " 	Unity中的四元数与欧拉角"
date:       2020-04-15 23:00:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 数学
    - 游戏数学
---

## 游戏开发中，数学的应用是非常广泛的，本文主要总结一些平时常用的游戏数学知识

### 一.欧拉角

> 欧拉角是用来唯一地确定定点转动刚体位置的三个一组独立角参量。

> Unity中的欧拉角有两种方式可以解释:   
  1,当认为顺序是yxz时（其实就是heading - pitch - bank），是传统的欧拉角变换，也就是以物体自己的坐标系为轴的。  
  2,当认为顺序是zxy时（roll - pitch - yaw），也是官方文档的顺序时，是以惯性坐标系为轴的。后者比较直观一些，但其实两者的实际效果是一样的，只是理解不一样。


```cs
    //Uniy中，最直接的欧拉角旋转方式
    Transform A;
    A.Rotate(new Vector3(0, 30, 0));
    //或者
    A.eulerAngles = new Vector3(0.0f, 10.0f, 0.0f);
```

使用欧拉角旋转，由于旋转顺序的原因，容易产生万向锁。
> 万向锁的意思就是，在三维旋转过程中，会丢失一个纬度，造成两个纬度的旋转，效果确实绕着同一个轴。

Unity中最简单的万向锁就是先让X轴旋转90度，此时Z轴与物体旋转为(0,0,0)时候的Y轴(也是惯性坐标系Y轴)重合了。

> 比较经典的一个欧拉角万向锁的动画演示视频: https://v.youku.com/v_show/id_XNzkyOTIyMTI=.html

![avatar](https://img-blog.csdn.net/20160223115811920?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 二.四元数

> Quaternion又称四元数，由x,y,z和w这四个分量组成，是由爱尔兰数学家威廉·卢云·哈密顿在1843年发现的数学概念。四元数的乘法不符合交换律。从明确地角度而言，四元数是复数的不可交换延伸。如把四元数的集合考虑成多维实数空间的话，四元数就代表着一个四维空间，相对于复数为二维空间。

四元数相对于直接使用欧拉角的好处是不会产生万向锁。

```cs
    //Unity

    Transform A;
    Quaternion rotations = Quaternion.identity;
    rotations.eulerAngles = new Vector3(0.0f, tSpeed, 0.0f);
    A.rotation = rotations;

```

大部分情况可能只会用到如下的方法:

```cs

  Quaternion.LookRotation
  Quaternion.Angle
  Quaternion.Euler
  Quaternion.Slerp
  Quaternion.FromToRotation
  Quaternion.identity

```

> 总结: Quaternion是基于复数，并不容易直观地理解,而且几乎不需要访问或修改单个四元数参数（x，y，z，w),所以只要了解四元数存在的意义就可以了。