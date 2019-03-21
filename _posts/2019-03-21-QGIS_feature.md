---
layout:     post                    # 使用的布局（不需要改）
title:      QGIS地理实体抽象               # 标题 
subtitle:   
date:       2019-03-21              # 时间
author:     Kelly Yang                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - QGIS
    - QgsGeometry
---

# 概述
参考：https://blog.csdn.net/sf2gis2/article/details/39649599

地理实体抽象是指点、线、面及其组合而成的，用于描述实际地物的数据结构。

其中包含几何实体和属性数据。

GIS中进行几何操作，以各种实体类为基础进行操作。

在OGC中，地理实体可以由WKT表示。在Qgis中使用GEOS和WKT两种方式表示，并在逐步将GEOS全部转化为WKT表示。

在内存中，以WKB格式存储。

WKT:使用文本存储几何对象。

WKB：使用结构进行存储。

属性数据一般与几何数据分开存储，两者通过相应的id进行联系，属性数据在QGIS中使用QVector进行存储。

# 方法
QGIS中的将所有的类型，以QgsVector<T>为模板，以QgsPoint为基础进行组织实现,QgsRectangle单独实现。

QgsVector<T>：向量抽象，实现通用功能，如向量-*/，旋转等操作。在QgsPoint.h中实现。

QgsPoint:点抽象，实现点的功能。如：距离，方位角，运算，测试与线的关系等。

QgsRectangle：矩形抽象，实现缩放，测试（包含等），合并，融合等。

/** polyline is represented as a vector of points */多段线也就是点集合
typedef QVector<QgsPoint> QgsPolyline;

/** polygon: first item of the list is outer ring, innerrings (if any) start from second item */
typedef QVector<QgsPolyline> QgsPolygon;

/** a collection of QgsPoints that share a common collection of attributes */
typedef QVector<QgsPoint> QgsMultiPoint;

/** a collection of QgsPolylines that share a commoncollection of attributes */
typedef QVector<QgsPolyline>QgsMultiPolyline;

/** a collection of QgsPolygons that share a commoncollection of attributes */
typedef QVector<QgsPolygon>QgsMultiPolygon;

**QgsGeometry:所有实体的抽象，用于将上述所有实现进行几何操作的统一平台。并与GEOS库交互，进行几何分析功能。**

# 带有属性的地理实体抽象QgsFeature

QgsFeature用于抽象一个带有属性的地理实体，由其联系几何和属性两部分。几何部分由QgsGeometry抽象。属性部分由QgsFields和QgsAttributes抽象。

QgsField:属性名抽象，可以操作属性名的各个成员。

QgsFields：是QgsField的集合操作类。

QgsFeatureIds:QgsFeatureId的集合。

QgsFeatureId:用于唯一标准一个QgsFeature的id。当前是64位Int。

QgsFeatureMap: typedef QMap<QgsFeatureId,QgsFeature> QgsFeatureMap;

QgsGeometryMap: typedef QMap<QgsFeatureId,QgsGeometry> QgsGeometryMap;

QgsAttributes：QVector<QVarient>，表示属性的值。

QgsAbstractFeatureIterator: QgsFeature迭代器的虚基类，由DataProvider驱动实现。用于进行元素获取。

QgsFeatureIterator：QgsFeature迭代器的包装类，用于操作QgsAbstractFeatureIterator。
（也就是操作的时候用这个）

QgsAbstractFeatureSource:由驱动实现，提供元素获取。

QgsAbstractFeatureFromSource<T>:抽象模板，继承QgsAbstractFeatureIterator，操作QgsAbstractFeatureSource。用于获取驱动（T），并进行读写开关操作（QgsAbstractFeatureIterator）。

QgsVectorLayerFeatureIterator:QgsFeature迭代器的矢量图层实现,每个矢量图层必须实现本类。

# 相关类
QgsGeometry 见同系列QgsGeometry文章

