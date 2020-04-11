---
layout:     post
title:      "「Egret」6 2D游戏的动态换装--换皮肤"
subtitle:   " 	图集的动态生成,整体换肤实现动态换装"
date:       2020-04-11 23:00:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏引擎
    - 动态换装
    - Egret
---

### Egret 中的动态图集生成

## 1.为什么要生成动态图集

> 在游戏制作中，经常遇到换装的需求，对于2d游戏来说，换装有很多局限性，本文是基于dragonbones实现的换肤换装。

> 这里不考虑单独替换部位实现换装，只探讨在使用皮肤换装的情况下如何更方便的实现动态样式的换装。

## 2.直接的皮肤换装

```ts
    private replaceSkin() {
        let tex = RES.getRes("skin1_png");
        this.armature.armature.replaceTexture(tex);
    }
```
> 这样是最简单的换装，可以制作多张同样大小的图集，每次换装直接使用新的皮肤图集替换，缺点也很明显，无法动态替换某些部位的贴图。

## 3.只有一张默认皮肤+想换掉其中的武器

> 使用皮肤换装，又想换掉其中的武器部位，那么只能将武器部位替换之后，生成新的图集，再来进行整体的皮肤换装。


```ts
  private generateNewTexture() {
        let basicSkinTexName = "basicSkin_tex_png";
        let weaponTexName = "weapon_tex_png";

        //基础的皮肤贴图图集
        let basicSkinTexAtlas = RES.getRes("basicSkin_tex_json");
        //基础的皮肤图集中武器部位的贴图名字        
        let basicWeaponName = "weapon";

        //武器贴图图集      
        let weaponTexAtlas = RES.getRes("weaponTex_tex_json");
        //要替换的武器在图集中的名字
        let replaceWeaponName = "weapon_big";

        //新图集贴图的容器
        var sp = new egret.DisplayObjectContainer;
        sp.width = 512;
        sp.height = 1024;

        var subTexture = <Array<SkeAtlasData>>basicSkinTexAtlas.SubTexture;
        for (let frame of subTexture) {
            let frameName: string = frame.name;
            frameName = frameName.slice(3);
            //用于在内存中缓存的名字
            var targetPNGName = "";
            let img = null;

            if (frameName == basicWeaponName) {
                targetPNGName = replaceWeaponName + "_png";
                let weaponRect = this.getSheetRectByName(weaponTexAtlas, replaceWeaponName);
                img = this.getSheetTexture(weaponTexName, targetPNGName, new egret.Rectangle(frame.x, frame.y, frame.width, frame.height));
            } else {
                targetPNGName = frameName + "_png";
                img = this.getSheetTexture(basicSkinTexName, targetPNGName, new egret.Rectangle(frame.x, frame.y, frame.width, frame.height));
            }

            if (img) {
                //将得到的贴图放到新的容器中，按照皮肤图集中贴图的位置摆放
                let bbb = new egret.Bitmap();
                bbb.texture = img;
                let rt = bbb;
                sp.addChild(rt);
                rt.x = frame.x;
                rt.y = frame.y;
            } else {
                egret.log("no image : " + targetPNGName);
            }
        }


        var rt1 = new egret.RenderTexture();
        var rt2 = new egret.RenderTexture();
        //将贴图容器渲染成新的贴图,此贴图为倒影，不能直接使用
        rt1.drawToTexture(sp, new egret.Rectangle(0, 0, 512, 1024), 1);
        rt1.bitmapData.webGLTexture = rt1.bitmapData.source.texture;
        rt1.bitmapData.webGLTexture.smoothing = true;
        rt1.bitmapData.source = null;
        let bitmap2: egret.Bitmap = new egret.Bitmap(rt1);
        //将贴图再次渲染，得到正确贴图
        rt2.drawToTexture(bitmap2, new egret.Rectangle(0, 0, 512, 1024), 1);
        rt2.bitmapData.webGLTexture = rt2.bitmapData.source.texture;
        rt2.bitmapData.webGLTexture.smoothing = true;
        rt2.bitmapData.source = null;
        rt1.dispose();

        return rt2;
    }

    public getSheetTexture(sheetName: string, textureName: string, rect: egret.Rectangle): egret.Texture {
        let sheet: egret.SpriteSheet = null;
        if (sheet == null) {
            let res = RES.getRes(sheetName);
            if (res == null) {
                return null;
            }
            sheet = new egret.SpriteSheet(res);
        }

        var source = sheet.getTexture(textureName);
        if (source == null) {
            source = sheet.createTexture(textureName, rect.x, rect.y, rect.width, rect.height);
        }
        return source;
    }

    public getSheetRectByName(atlas: any, name: string) {
        //根据图集的不同实现不同
    }
```

> 得到新的贴图，就可以调用replaceTexture整体换装了，这里只演示了单个武器的替换，实际情况中会有更复杂的情况，可能包含几十种贴图混合。

> 这里的武器图集也可以使用单张贴图，在这里用两个图集是因为会有两种皮肤混合使用的需求。

> 以上的方法都依托于基础的皮肤图集中，所有部位的大小要提前规定好，单个部位贴图可以设计的大一点，里面具体内容的填充度可以灵活处理，来实现同样部位，大小不同的效果。
  