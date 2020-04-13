---
layout:     post
title:      "「Egret」8 实现简单的翻书效果"
subtitle:   " 	简单的翻书效果"
date:       2020-04-13 23:00:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏引擎
    - Egret
    - 翻书效果
---

## 1.简介

> 首先感谢Active_Loser的[自定义View实战篇（二）实现小说翻页二 基本原理](https://www.jianshu.com/p/9417dec25f5e )

翻书是很常见的效果，但是在egret中，由于跨平台的需求，引擎对底层的接口过度封装，并没有开放出特别多的绘制函数，所以是表现出来的效果和直接使用canvas绘制还是有一些限制。

具体原理可以去看Active_Loser的原理介绍，图解的很详细，这里我就不放多余的图片了，原理其实很简单，就是将若干点连接到一起进行裁剪，显示出来的视差效果就会像翻书一样。

## 2.Egret中的实现

主要是提供在Egret中的实现思路，在保证跨平台的基础下实现效果，当然下面使用的mask其实效率是非常低的，所以在一般游戏中很少会这样使用。

```ts
class FlipBookTest extends egret.DisplayObjectContainer {
    constructor() {
        super();
        this.width = 640;
        this.height = 1136;
    }

    private img: eui.Image;

    public doTest() {
        let group = new eui.Group();
        group.width = this.width;
        group.height = this.height;
        this.addChild(group);
        this.init();

        // let img1 = new eui.Image();
        // img1.source = "WX20200413-222423@2x_png";
        // img1.width = this.width;
        // img1.height = this.height;
        // group.addChild(img1);

        this.img = new eui.Image();
        this.img.source = "WX20200413-221539@2x_png";
        this.img.width = this.width;
        this.img.height = this.height;
        group.addChild(this.img);
    }

    private mShapeA: egret.Shape;
    private mShapeB: egret.Shape;
    private mShapeC: egret.Shape;

    public freshView() {
        let sp = this.getShapeA();
        this.img.mask = sp;
        // this.getShapeC();
    }

    private a: egret.Point; f: egret.Point;
    g: egret.Point; e: egret.Point;
    h: egret.Point; c: egret.Point; j: egret.Point;
    b: egret.Point; k: egret.Point; d: egret.Point; i: egret.Point;

    private init() {
        this.a = new egret.Point();
        this.f = new egret.Point();
        this.g = new egret.Point();
        this.e = new egret.Point();
        this.h = new egret.Point();
        this.c = new egret.Point();
        this.j = new egret.Point();
        this.b = new egret.Point();
        this.k = new egret.Point();
        this.d = new egret.Point();
        this.i = new egret.Point();

        this.mShapeA = new egret.Shape();
        this.mShapeB = new egret.Shape();
        this.mShapeC = new egret.Shape();

        this.mShapeA.width = this.width;
        this.mShapeA.height = this.height;
        this.addChild(this.mShapeA);

        this.mShapeC.width = this.width;
        this.mShapeC.height = this.height;
        this.addChild(this.mShapeC);

        this.f.x = this.width;
        this.f.y = this.height;

        this.addEventListener(egret.TouchEvent.TOUCH_BEGIN, this.onTouchBegin, this);
        this.addEventListener(egret.TouchEvent.TOUCH_MOVE, this.onTouchMoveEvent, this);
        this.addEventListener(egret.TouchEvent.TOUCH_END, this.onTouchEnd, this);
    }

    public onTouchBegin(event: egret.TouchEvent) {
        egret.log("onTouchBegin");
    }

    public onTouchMoveEvent(event: egret.TouchEvent) {
        let x = event.$stageX;
        let y = event.$stageY;
        this.a.x = x;
        this.a.y = y;
        this.calculationPoint(this.a, this.f);
        if (this.c.x < 0) {
            this.calculationAByYouch();
            this.calculationPoint(this.a, this.f);
        }
        this.freshView();
    }

    public onTouchEnd(event: egret.TouchEvent) {
        egret.log("onTouchEnd");
    }

    /**
    * 获取区域A的shape
    */
    private getShapeA() {
        this.mShapeA.graphics.clear();
        this.mShapeA.graphics.beginFill(0);
        this.mShapeA.graphics.lineStyle(1, 0xff0000);
        this.mShapeA.graphics.lineTo(0, this.height);
        this.mShapeA.graphics.lineTo(this.c.x, this.c.y);
        this.mShapeA.graphics.curveTo(this.e.x, this.e.y, this.b.x, this.b.y);
        this.mShapeA.graphics.lineTo(this.a.x, this.a.y);
        this.mShapeA.graphics.lineTo(this.k.x, this.k.y);
        this.mShapeA.graphics.curveTo(this.h.x, this.h.y, this.j.x, this.j.y);
        this.mShapeA.graphics.lineTo(this.width, 0);
        this.mShapeA.graphics.lineTo(0, 0);
        this.mShapeA.graphics.endFill();
        return this.mShapeA;
    }

    /**
     * 获取区域C的Shape
     */
    private getShapeC() {
        this.mShapeC.graphics.clear();
        this.mShapeC.graphics.beginFill(0);
        this.mShapeC.graphics.moveTo(this.i.x, this.i.y);
        this.mShapeC.graphics.lineTo(this.d.x, this.d.y);
        this.mShapeC.graphics.lineTo(this.b.x, this.b.y);
        this.mShapeC.graphics.lineTo(this.a.x, this.a.y);
        this.mShapeC.graphics.lineTo(this.k.x, this.k.y);
        this.mShapeC.graphics.endFill();
        return this.mShapeC.graphics;
    }

    /**
     * 获取区域B的Shape
     */
    private getShapeB() {
        this.mShapeB.graphics.clear();
        this.mShapeB.graphics.beginFill(0);
        this.mShapeB.graphics.lineTo(0, this.height);
        this.mShapeB.graphics.lineTo(this.width, this.height);
        this.mShapeB.graphics.lineTo(this.width, 0);
        this.mShapeB.graphics.endFill();
        return this.mShapeB.graphics;
    }

    /**
     * 计算各个点的坐标
     *
     * @param a a点的坐标
     * @param f f点的坐标
     */
    private calculationPoint(a: egret.Point, f: egret.Point) {
        this.g.x = (a.x + f.x) / 2;
        this.g.y = (a.y + f.y) / 2;

        this.e.x = this.g.x - (f.y - this.g.y) * (f.y - this.g.y) / (f.x - this.g.x);
        this.e.y = f.y;

        this.h.x = f.x;
        this.h.y = this.g.y - (f.x - this.g.x) * ((f.x - this.g.x) / (f.y - this.g.y));

        this.c.x = this.e.x - (f.x - this.e.x) / 2;
        this.c.y = f.y;

        this.j.x = f.x;
        this.j.y = this.h.y - (f.y - this.h.y) / 2;

        this.b.x = (a.x + this.e.x) / 2;
        this.b.y = (a.y + this.e.y) / 2;

        this.k.x = (a.x + this.h.x) / 2;
        this.k.y = (a.y + this.h.y) / 2;

        this.d.x = ((this.c.x + this.b.x) / 2 + this.e.x) / 2;
        this.d.y = ((this.c.y + this.b.y) / 2 + this.e.y) / 2;

        this.i.x = ((this.k.x + this.j.x) / 2 + this.h.x) / 2;
        this.i.y = ((this.k.y + this.j.y) / 2 + this.h.y) / 2;
    }

    private calculationAByYouch() {
        let cf = this.width - this.c.x;

        let pf = Math.abs(this.f.x - this.a.x);
        let p1f = this.width * pf / cf;
        this.a.x = Math.abs(this.f.x - p1f);

        let h1 = Math.abs(this.f.y - this.a.y);
        let a1p1 = h1 * pf / cf;
        this.a.y = Math.abs(this.f.y - a1p1);
    }

}
```

![avatar](http://q8ixw72rd.bkt.clouddn.com/2020-04-13-egret-flipbook.gif)