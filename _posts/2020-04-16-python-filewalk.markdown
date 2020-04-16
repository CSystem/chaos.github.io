---
layout:     post
title:      "「Python」 自动化图片压缩处理(TinyPng)"
subtitle:   " 	使用Python实现自动化图片压缩处理"
date:       2020-04-16 23:00:00
author:     "AF"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Python
    - 图片压缩
---

## web或移动端项目中，经常遇到贴图资源太大，需要压缩处理的问题，这里使用Tiny作为压缩方式实现

### 1.tinify压缩

> tinify是TinyPng提供的Python的压缩库

```python
import tinify
#存放多个免费的key
tinifyKeys = ["key","key"]
#当前使用到具体的key的idx
currentKeyIdx = 0

#账号压缩张数超过一定数量就切换下一个账号
def tinycompress(filepath):
    #tiny官网注册账号，可以获得key，每个月500张的压缩额度，如果想免费试用，就多申请几个账号
    tinify.key = tinifyKeys[currentKeyIdx]
    source = tinify.from_file(filepath)
    source.to_file(filepath)

    if tinify.compression_count >= 495:
        global currentKeyIdx
        currentKeyIdx += 1
        print("current keyid ： " + str(currentKeyIdx))
        if currentKeyIdx >= len(tinifyKeys):
            print("no available key")
            return
        tinify.key = tinifyKeys[currentKeyIdx]
```

### 2.PIL裁剪

> 可以使用PIL库，限制图片最大的尺寸，因为在某些设备上，对贴图最大尺寸有要求，用这个库来等比缩放

```python

import os, sys, io
from PIL import Image

def resizeFilter(filename):
    if ".png" in filename:
        return True
    if ".jpg" in filename:
        return True
    return False

def resize(dirpath,maxsize):
    for parent, dirnames, filenames in os.walk(dirpath):
        for filename in filenames:
            if not resizeFilter(filename):
                continue
            filepath = parent + "/" + filename
            im = Image.open(filepath)
            (x, y) = im.size
            if y <= maxsize and x <= maxsize:
                continue

            if x > y :
                width = maxsize
                height = int(maxsize / x * y)
            else :
                width = int(maxsize / y * x)
                height = maxsize
            out = im.resize((width, height), Image.ANTIALIAS)
            out.save(filepath)
```

### 3.记录文件压缩状态

```python

rootDir = "../resource/assets/"  # 资源根目录
hashmapDir = "../resource/compressHashMap.json"

def initHashMap():
    if not os.path.exists(hashmapDir):
        print("compressHashMap 不存在 : 自动创建")
        hashfile = open(hashmapDir, "w+", encoding='utf8')
        hashfile.write(json.dumps({}))
        hashfile.close()


# 更新hashmap，当工程文件出现变动时，标记变动的文件为需要压缩的
# 只处理图片.jpg,.png
def updateHashMap():
    hashfile = open(hashmapDir, "r", encoding='utf8')
    hashrecord = json.load(hashfile)
    hashfile.close()
    hashmap = {}
    compress_tag = {}

    if ("filelist" in hashrecord):
        hashmap = hashrecord["filelist"]

    if ("tag" in hashrecord):
        compress_tag = hashrecord["tag"]

    for parent, dirnames, filenames in os.walk(rootDir):
            for filename in filenames:
                if not compressFilter(filename):
                    continue
                fileSrcPath = parent + "/" + filename
                (hvalue, ctimes) = file_md5(fileSrcPath)
                # 如果hash表中不存在，说明是新增资源
                if filename not in hashmap:
                    hashmap[filename] = hvalue
                    print("file is new,need to recompress >> " + filename)
                    compress_tag[filename] = fileSrcPath
                else:
                    oldHValue = hashmap[filename]
                    if oldHValue != hvalue:
                        hashmap[filename] = hvalue
                        print("file is old,need to recompress >> " + filename)
                        compress_tag[filename] = fileSrcPath
    hashrecord["filelist"] = hashmap
    hashrecord["tag"] = compress_tag
    hashfile = open(hashmapDir, "w+", encoding='utf8')
    hashfile.write(json.dumps(hashrecord))
    hashfile.close()

```

### 4.检查文件状态并压缩

```python

import os, sys, io
import json
import tinify
import shutil
import hashlib

def compress():
    hashfile = open(hashmapDir, "r", encoding='utf8')
    hashrecord = json.load(hashfile)
    hashfile.close()
    hashmap = hashrecord["filelist"]
    compress_tag = {}

    if ("tag" in hashrecord):
        compress_tag = hashrecord["tag"]
    complete_list = []
    for filename in compress_tag:
        filepath = compress_tag[filename]
        print("start compress : " filename)
        tinycompress(filepath)
        (hvalue, ctimes) = file_md5(filepath)
        hashmap[filename] = hvalue
        complete_list.append(filename)
        print("compress end : " filename)
     
    for i in range(len(complete_list)):
        filename = complete_list[i]
        if filename in compress_tag:
            del compress_tag[filename]

    hashrecord["filelist"] = hashmap
    hashrecord["tag"] = compress_tag
    hashfile = open(hashmapDir, "w+", encoding='utf8')
    hashfile.write(json.dumps(hashrecord))
    hashfile.close()

```

### 5.合并

```python

def normalcompress():
    #先进行缩放
    resize(rootDir,2048)
    #检查文件状态，是否有需要重新压缩的资源
    updateHashMap()
    #开始根据文件状态压缩
    compress()

if __name__ == "__main__":
    initHashMap()
    normalcompress()

```

> 将文件状态检测和压缩分开是因为，压缩过程是同步阻塞过程，如果放在检查文件状态中，容易造成过程中断，影响对所有资源的宏观管理