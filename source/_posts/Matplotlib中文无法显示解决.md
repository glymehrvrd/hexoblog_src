---
title: Matplotlib中文无法显示解决
date: 2017-01-03 10:13:35
tags:
- Data Science
- Python
- Matplotlib
---

`Matplotlib`默认配置中，中文会显示为`口口`。

<!-- more -->

网上的教程中的方法，修改matplotlibrc文件，`python\Lib\site-packages\matplotlib\mpl-data`。

修改以下内容
```
font.family     : Microsoft YaHei

font.sans-serif : Microsoft YaHei, Bitstream Vera Sans, Lucida Grande, Verdana, Geneva, Lucid, Arial, Helvetica, Avant Garde, sans-serif
```

即去掉注释，并在配置值中添加 Microsoft YaHei。然后在windows下搜索msyh.ttf，即微软的雅黑字体，
并将msyh.ttf拷贝到目录`python\Lib\site-packages\matplotlib\mpl-data\fonts\ttf`中。

但是我做了依然不行，最后在stackoverflow上找到答案。

> When you add new fonts to your system, you need to delete your fontList.cache file in order for matplotlib to find them.

> The reason it works on lines 4/5 in your example is because you are creating a FontManager from scratch (which goes out to the filesystem and hunts down all the fonts). Internally, when matplotlib later does its own font lookup, it is using a FontManager that has been loaded from a cache on disk in the fontList.cache file.

> Long term, we have plans to switch to using the font lookup mechanisms of the OS to get around this problem, (see MEP14), but in the meantime, you'll need to remove the fontList.cache file everytime you want matplotlib to discover new fonts.

即需要删除`C:\Users\yourUsername\.matplotlib\fontList.cache`，然后重新import matplotlib让它重新扫描所有字体并缓存。