---
layout:     post
title:      "「游戏中的数学」 1 向量"
subtitle:   " 	2D,3D游戏数学"
date:       2020-04-13 23:00:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 数学
    - 游戏数学
---

## 游戏开发中，数学的应用是非常广泛的，本文主要总结一些平时常用的游戏数学知识

### 一.向量 Vector

> 向量（也称为欧几里得向量、几何向量、矢量），指具有大小（magnitude）和方向的量。在游戏开发中它经常用来描述位置的变化，并且可以与其他向量相加或者相减来得到新的位置变化（一个向量代表一个位置的变化，两个这样的向量相加得到的是这两段位置变化的总效果）。向量根据坐标系的纬度分为Vector2(x,y)和Vector3(x,y,z)两种向量,还有一种四维向量Vector4(x,y,z,w)一般应用于着色器或网格渲染中，w代表特殊的含义。

#### 1.向量的加减法

> 向量加法满足平行四边形法则，即相加后的向量等于对角线向量,在unity中物体的position就是用vector来表示的。

 v1(x,y) + v2(x,y) = v3(v1.x + v2.x , v1.y + v2.y)

 v1(x,y) - v2(x,y) = v3(v1.x - v2.x , v1.y - v2.y)

``` csharp
transform.position += Vector3.one*Time.deltatime
```

> 向量的减法满足三角形法则，即相减后的向量就是两向量围成三角的第三边，方向指向被减向量。在unity中最常见的用于获取方向、向量长度等。

```csharp
Vector3 direction = (Position1 - Position2).normalized;  //标准化向量
transform.Translate(direction*0.2f*Time.deltaTime);
```

> Vector.normalized 方向向量的一个特例是长度为1的向量。它也被称为归一化的向量或者简称为标准向量。

#### 2.向量的点乘(dot)

> 在游戏中利用点乘可判断一个多边形是否面向摄像机还是背向摄像机。向量的点积与它们夹角的余弦成正比，因此在聚光灯的效果计算中，可以根据点积来得到光照效果，如果点积越大，说明夹角越小，则物理离光照的轴线越近，光照越强。

v1( x1, y1，z1) * v2(x2, y2,z2) = x1 * x2 + y1 * y2 + z1 * z2;

如果点积返回的结果为1，说明这两个法向量指向同一个方向。

如果点积返回的结果为0，说明这两个法向量互相垂直。

如果点积返回的结果为-1，说明这两个法向量指向完全相反的方向。

下面这张图说明的是点积返回的结果与两个向量之间夹角的关系：

![avatar](http://q8ixw72rd.bkt.clouddn.com/2020-04-14-gamemath-vector-1.jpg)

angle = acos(A dot B)

```cs
    //Unity

    // 计算 a、b 点积结果
    float result = Vector3.Dot(a, b);
    // 通过向量直接获取两个向量的夹角（默认为 角度）， 此方法范围 [0 - 180]
    float angle = Vector3.Angle(a, b);
    // 计算 a、b 单位向量的点积,得到夹角余弦值,|a.normalized|*|b.normalized|=1;
    result = Vector3.Dot(a.normalized, b.normalized);
    // 通过反余弦函数获取 向量 a、b 夹角（默认为 弧度）
    float radians = Mathf.Acos(result);
     // 将弧度转换为 角度
    angle = radians * Mathf.Rad2Deg;
    

```

### 3.向量的叉乘(cross)

向量叉乘也是对两个向量的一个操作。结果是一个新的向量,同时也称作法向量,它与前两个向量垂直，并且它长度是前两个向量长度的均值,这个向量的模是以两个向量为边的平行四边形的面积。

v1( x1, y1，z1) x v2(x2, y2, z2) = (y1*z2 - y2*z1)i+(x2*z1 - x1*z2)j+(x1*y2-x2*y1)k

> |aXb| = |a|*|b|*sin<θ>

> θ = ARCSin（|aXb|）

右手法则：右手的四指方向指向第一个矢量,屈向叉乘矢量的夹角方向（两个矢量夹角方向取小于180°的方向）,那么此时大拇指方向就是叉乘所得的叉乘矢量的方向.（大拇指应与食指成九十度）（注意：Unity当中使用左手，因为Unity使用的是左手坐标系）

unity中的左手法则
![avatar](http://gameweb-img.qq.com/gad/20170220/phpstyGpN.1487576939.png)

```cs
    //unity

    //计算向量 a、b 的叉积，结果为 向量 
    Vector3 c = Vector3.Cross(a, b);
 
    // 通过反正弦函数获取向量 a、b 夹角（默认为弧度）
    float radians = Mathf.Asin(Vector3.Distance(Vector3.zero, Vector3.Cross(a.normalized, b.normalized)));
    float angle = radians * Mathf.Rad2Deg;
 
    // 判断顺时针、逆时针方向，是在 2D 平面内的，所以需指定一个平面，
    //下面以X、Z轴组成的平面为例 , (Y 轴为纵轴),
    // 在 X、Z 轴平面上，判断 b 在 a 的顺时针或者逆时针方向,
     if (c.y > 0)
    {
        // b 在 a 的顺时针方向
    }
    else if (c.y == 0)
    {
        // b 和 a 方向相同（平行）
    }
    else
    {
        // b 在 a 的逆时针方向
    }
```

## 4.向量的模(长度)

²√x²+y²+z²


> 总结:常规3D游戏引擎中使用的都是左手坐标系。向量的加减主要用于移动以及距离的计算，点乘判断方向，叉乘判断角度。因为点乘角度范围是0-180度，所以需要结合叉乘来计算旋转的方向。