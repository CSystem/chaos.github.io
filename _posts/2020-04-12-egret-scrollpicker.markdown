---
layout:     post
title:      "「Egret」7 滑动列表选择器"
subtitle:   " 	用于时间，日期，或者其他元素的滑动定位选择"
date:       2020-04-12 23:00:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏引擎
    - 滑动选择器
    - Egret
---

### Egret 中简单的滑动列表选择器

    主要用在类似移动端的日期选择器或者一个列表，需要定位到某个元素位置的情况。

> 使用系统的eui.sys.TouchScroll组件，覆盖scroll的事件来实现定位滑动。

```ts
class ScrollPicker extends egret.EventDispatcher {
    /**
     * scrollPolicy : ScrollPicker.ScrollPolicyH or ScrollPicker.ScrollPolicyV
     */
    constructor(scroll: eui.Scroller, selectedIdx: number, itemSize: number, scrollPolicy: string) {
        super();
        this._scroller = scroll;
        this._group = scroll.viewport as eui.DataGroup;
        this._selectedIdx = selectedIdx;
        this._itemSize = itemSize;
        this._scroller.scrollPolicyH = "off";
        this._scroller.scrollPolicyV = "off";
        this._scrollPolicy = scrollPolicy;
        this.init();
    }

    private _scroller: eui.Scroller;
    private _group: eui.DataGroup;
    private _scrollPolicy: string;
    private _scrollMax: number = 0;
    private _itemSize: number = 200;
    private _touchScroll: eui.sys.TouchScroll;
    private _selectedIdx = 0;
    private _currentSelectIdx = 2;

    public static ScrollPolicyH = "policyH";
    public static ScrollPolicyV = "policyV";

    public getCurrentItem(): any {
        var tmp = this.getScrollValue() / this._itemSize + this._selectedIdx;
        if (tmp >= this._group.dataProvider.length)
            tmp = this._group.dataProvider.length - 1;
        if (tmp < 0)
            tmp = 0;

        var value = this._group.dataProvider.getItemAt(this._currentSelectIdx);
        return value;
    }

    public setCurrentSelected(idx: number) {
        let val = this._itemSize * (idx);
        this._scroller.viewport.scrollV = val;
    }

    private init(): void {
        this._scrollMax = (this._group.dataProvider.length - 1 - this._selectedIdx) * (this._itemSize);
        this._group.addEventListener(egret.TouchEvent.TOUCH_BEGIN, this.touchBegin, this);
        this._touchScroll = new eui.sys.TouchScroll(this.updateFunction, this.endFunction, this);
        this._touchScroll.$scrollFactor = 5;
        this._touchScroll.$bounces = false;
        this._scroller.throwSpeed = 0;
    }

    private updateFunction(scrollPos: number): void {
        if (this._scrollPolicy == ScrollPicker.ScrollPolicyH) {
            this._scroller.viewport.scrollH = scrollPos;
        } else {
            this._scroller.viewport.scrollV = scrollPos;
        }
    }

    private endFunction(): void {
        var f = this.getScrollValue() / (this._itemSize);
        var tmp = Math.floor(f);
        var idx = 0;
        if (f - tmp > 0.5) {
            idx = tmp + 1;
        } else {
            idx = tmp;
        }

        var scroll = (this._itemSize) * idx;
        if (this._scrollPolicy == ScrollPicker.ScrollPolicyH) {
            egret.Tween.get(this._scroller.viewport).to({ scrollH: scroll }, 100);
        } else {
            egret.Tween.get(this._scroller.viewport).to({ scrollV: scroll }, 100);
        }
        this._scroller.viewport
        this._currentSelectIdx = idx + this._selectedIdx;
        this.disPatchValueChangeEvent();
    }

    private _lastPosX: number = 0;
    private _lastPosY: number = 0;
    private touchBegin(evt: egret.TouchEvent) {
        this._lastPosX = evt.stageX;
        this._lastPosY = evt.stageY;
        this._touchScroll.stop();
        this._touchScroll.start(this._lastPosY);
        egret.MainContext.instance.stage.addEventListener(egret.TouchEvent.TOUCH_END, this.touchEnd, this);
        egret.MainContext.instance.stage.addEventListener(egret.TouchEvent.TOUCH_MOVE, this.touchMoved, this);
    }

    private touchEnd(event: egret.TouchEvent) {
        if (this._touchScroll.isStarted()) {
            this._touchScroll.finish(this.getScrollValue(), this._scrollMax);
        }
        egret.MainContext.instance.stage.removeEventListener(egret.TouchEvent.TOUCH_END, this.touchEnd, this);
        egret.MainContext.instance.stage.removeEventListener(egret.TouchEvent.TOUCH_MOVE, this.touchMoved, this);
    }

    private touchMoved(event: egret.TouchEvent) {
        this._touchScroll.update(event.$stageY, this._scrollMax, this.getScrollValue());
    }

    private disPatchValueChangeEvent() {
        let currentIdx = this.getCurrentItem();
        egret.log("停在了 :" + currentIdx);
    }

    private getScrollValue(): number {
        if (this._scrollPolicy == ScrollPicker.ScrollPolicyH) {
            return this._scroller.viewport.scrollH;
        }
        return this._scroller.viewport.scrollV;
    }
}
```

![avatar](http://www.57wan8.com/2020-04-12-egret-scrollpicker.gif)