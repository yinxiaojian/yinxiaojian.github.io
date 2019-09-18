---
layout: title
title: google map 比例尺算法分析
date: 2017-12-24 13:26:14
tags: google map
categories: 前端
---

比例尺即地图右下角显示地图距离与实际距离比例的控件，由于Google map自带比例尺控件存在的局限性——无法调整位置和格式，所以通过此文章，介绍比例尺算法及具体实现。

<!-- more -->

### 什么是比例尺

比例尺是表示[图上距离](http://baike.baidu.com/view/5454725.htm)比实地距离缩小的程度，因此也叫[缩尺](http://baike.baidu.com/view/1559050.htm)。用公式表示为：比例尺=[图上距离](http://baike.baidu.com/view/5454725.htm)/实地距离。在Google map上比例尺显示在右下角。

在创建地图时只需增加设置项

```
scaleControl: true
```

如下例

{% jsfiddle yinxiaojian/yL9mxdzp %}

遗憾的是这样添加的比例尺会存在于右下角，而且不像其他控件一样可以调整位置。如果我们希望修改其位置或者样式，就会无从下手。

### 自制比例尺

实现一个比例尺的关键在于如何获取到地图距离与实际距离的比例和缩放等级及维度之间的关系。google官方api未提供相关函数，因此我们需要自己计算。核心公式为

```
ScaleValue = 156543.03392 * Math.cos(latLng.lat() * Math.PI / 180) / Math.pow(2, zoom)
```

其中zoom为当前缩放等级，latLng.lat()即目标点维度值。该公式是在地球半径为6378137m的基础上计算的，这个值即google地图所采用的值。

有了计算公式后，我们还需要一张表——缩放等级和比例尺对应表，也就是在什么样的缩放等级下使用多大的比例尺，表格如下:

```
Zoom    Scale
0    10000km
1    5000km
2    2000km
3    1000km
4    500km
5    200km
6    200km
7    100km
8    50km
9    20km
10   10km
11   5km
12   2km
13   1km
14   500m
15   200m
16   200m
17   100m
18   50m
19   20m
20   10m
21   5m
22   2m
23   1m
24   1m
25   1m
26   1m
```

通过监听地图变化事件（缩放和平移），获取当前屏幕中心点缩放等级和维度获取到scale和scalevalue，那么比例尺的长度（px） = scale/scalevalue。

获取当前比例尺长度的核心代码如下：

```javascript
/**
 * 根据缩放等级和维度获取KM数(m数)和像素
 */
function setScaleInfos(zoomLevel, lat, map) {
	// 缩放等级-比例尺
	var zoomList = [{
		text: "10000KM",
		value: 10000 * 1000
	}, {
		text: "5000KM",
		value: 5000 * 1000
	}, {
		text: "2000KM",
		value: 2000 * 1000
	}, {
		text: "1000KM",
		value: 1000 * 1000
	}, {
		text: "500KM",
		value: 500 * 1000
	}, {
		text: "200KM",
		value: 200 * 1000
	}, {
		text: "200KM",
		value: 200 * 1000
	}, {
		text: "100KM",
		value: 100 * 1000
	}, {
		text: "50KM",
		value: 50 * 1000
	}, {
		text: "20KM",
		value: 20 * 1000
	}, {
		text: "10KM",
		value: 10 * 1000
	}, {
		text: "5KM",
		value: 5000
	}, {
		text: "2KM",
		value: 2000
	}, {
		text: "1KM",
		value: 1000
	}, {
		text: "500m",
		value: 500
	}, {
		text: "200m",
		value: 200
	}, {
		text: "200m",
		value: 200
	}, {
		text: "100m",
		value: 100
	}, {
		text: "50m",
		value: 50
	}, {
		text: "20m",
		value: 20
	}, {
		text: "10m",
		value: 10
	}, {
		text: "5m",
		value: 5
	}, {
		text: "2m",
		value: 2
	}, {
		text: "1m",
		value: 1
	}, {
		text: "1m",
		value: 1
	}, {
		text: "1m",
		value: 1
	}, {
		text: "1m",
		value: 1
	}];
	// 宽度
	var pxValue = Math.floor(zoomList[zoomLevel].value / (156543.03392 * Math.cos(lat * Math.PI / 180) / Math.pow(2, zoomLevel)));
	// 更新经纬度数据
	$W.id("scaleText").innerHTML = zoomList[zoomLevel].text;
	$W.id("scaleSize").style.width = pxValue + "px";
};
```

下面是通过上述思路实现的例子，在地图右下角实现一个比例尺。

{% jsfiddle yinxiaojian/6eutbuyr %}