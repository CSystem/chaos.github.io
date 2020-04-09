---
layout:     post
title:      "「Egret」4 加载TiledMap"
subtitle:   " 	读取tiledmap并显示"
date:       2020-04-09 22:25:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏引擎
    - TiledMap
    - Egret
---

### Egret+TiledMap

## 1.Egret tiled库下载地址

>　<https://github.com/egret-labs/egret-game-library/tree/master/tiled>

* 1.将库文件放到工程根目录的libs目录下
* 修改 Egret 工程内根目录下的egretProperties.json，在modules下添加tiled模块

``` json
 {
      "name": "tiled",
      "path": "./libs/tiled"
 }
```

## 2.修改TiledMap文件

* 使用tiled自带的exmaple中的地图，如果没有可以去<https://github.com/bjorn/tiled>下载

* Egret中只能选择Base64(uncompressed)模式，不支持压缩格式所以要打开修改一下,然后重新导出或直接保存tmx文件

![avatar](http://q8ixw72rd.bkt.clouddn.com/2020-04-09-egret-tiledmap-1.png)

> 注意： Egret 中还不支持载入tsx文件,用记事本打开desert.tmx文件

 ```html
<tileset firstgid="1" source="desert.tsx"/>
```

修改为：

```html
<tileset firstgid="1" name="Desert" tilewidth="32" tileheight="32" spacing="1" margin="1" tilecount="48">
<image source="tmw_desert_spacing.png" width="265" height="199"/>
</tileset>
```

> 修改完后复制tmw_desert_spacing.png以及desert.tmx到游戏工程中

## 3.Egret中读取TiledMap文件

```typescript
class TiledMapTest extends egret.DisplayObjectContainer {
    /*设置请求*/
    private request: egret.HttpRequest;
    /*设置资源加载路径*/
    private url: string;

    public doTest() {
        /*初始化资源加载路径*/
        this.url = "resource/assets/tiled/desert.tmx";
        /*初始化请求*/
        this.request = new egret.HttpRequest();
        /*监听资源加载完成事件*/
        this.request.once(egret.Event.COMPLETE, this.onMapComplete, this);
        /*发送请求*/
        this.request.open(this.url, egret.HttpMethod.GET);
        this.request.send();

    }

    /*地图加载完成*/
    private onMapComplete(event: egret.Event) {
        /*获取到地图数据*/
        var data: any = egret.XML.parse(event.currentTarget.response);
        /*初始化地图*/
        var tmxTileMap: tiled.TMXTilemap = new tiled.TMXTilemap(2000, 2000, data, this.url);
        tmxTileMap.render();
        /*将地图添加到显示列表*/
        this.addChild(tmxTileMap);
    }
}

```

效果如下:

![avatar](http://q8ixw72rd.bkt.clouddn.com/2020-04-09-egret-tiledmap-2.png)
