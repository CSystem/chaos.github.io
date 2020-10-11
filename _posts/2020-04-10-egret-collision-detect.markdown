---
layout:     post
title:      "「Egret」5 碰撞检测"
subtitle:   " 	多边形以及圆形的碰撞检测"
date:       2020-04-10 22:25:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏引擎
    - Collision Detection
    - Egret
---

### Egret中的碰撞检测

## 1.矩形和矩形之间的碰撞

```typescript

public static hitTest(obj1:egret.DisplayObject,obj2:egret.DisplayObject):boolean
{
    var rect1: egret.Rectangle = obj1.getBounds();
    var rect2: egret.Rectangle = obj2.getBounds();
    rect1.x = obj1.x;
    rect1.y = obj1.y;
    rect2.x = obj2.x;
    rect2.y = obj2.y;
    return rect1.intersects(rect2);
}
```

> 直接利用两个egret.DisplayObject物体的Rectangle来判断

## 2.圆和圆之间的碰撞

```typescript

let fromX = 1;
let fromY = 1;
let toX = 10;
let toY = 10;
let from = new egret.Point(fromX,fromY);
let to = new egret.Point(toX,toY);
let distance = egret.Point.distance(from,to);
if( distance <= width1/2 + width2/2 ){
    //todo 检测到了碰撞
}

```

> 利用圆心之间的距离来判断

## 3.圆和矩形之间的碰撞

> 计算圆和矩形的碰撞可以把圆和矩形放到同一个坐标系，因为圆是可以随意旋转的，以圆形的中点为坐标系原点，只要保证坐标系的x,y轴和矩形的对应边是平行的就好

![avatar](http://www.57wan8.com/2020-04-09-egret-tiledmap-1.png)

```typescript

  public intersects(circlePoint: egret.Point, radius: number, rect: egret.Rectangle): boolean {
        //在一个象限内计算距离
        if (this.isSameQuadrant(circlePoint, rect)) {
            return egret.Point.distance(circlePoint, rect.topLeft) <= radius || egret.Point.distance(circlePoint, new egret.Point(rect.right, rect.top)) <= radius
                || egret.Point.distance(circlePoint, new egret.Point(rect.left, rect.bottom)) <= radius || egret.Point.distance(circlePoint, rect.bottomRight) <= radius;
        } else {
            //跨象限可以把圆调整为矩形判断
            let cRect = new egret.Rectangle(circlePoint.x - radius, circlePoint.y - radius, radius * 2, radius * 2);
            return cRect.intersects(rect);
        }
    }

    /**
     * 判断矩形是不是在一个象限内
     * cood : 坐标原点
     */
    public isSameQuadrant(cood: egret.Point, rect: egret.Rectangle): boolean {
        var coodX = cood.x;
        var coodY = cood.y;
        var xoA = rect.topLeft.x
            , yoA = rect.topLeft.y
            , xoB = rect.bottomRight.x
            , yoB = rect.bottomRight.y;

        if (xoA - coodX > 0 && xoB - coodX > 0) {
            if ((yoA - coodY > 0 && yoB - coodY > 0) || (yoA - coodY < 0 && yoB - coodY < 0)) {
                return true;
            }
            return false;
        } else if (xoA - coodX < 0 && xoB - coodX < 0) {
            if ((yoA - coodY > 0 && yoB - coodY > 0) || (yoA - coodY < 0 && yoB - coodY < 0)) {
                return true;
            }
            return false;
        } else {
            return false;
        }
    }
```
