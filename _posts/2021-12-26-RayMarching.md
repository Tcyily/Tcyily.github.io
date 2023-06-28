---
layout: post
title: "体渲染: RayMarhcing + SDF"
date:   2021-12-26
tags: [Graphics]
comments: true
author: Tcyily
toc: true 
---

# 体渲染分享

## 引入
![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/_%E5%BC%95%E5%85%A5-%E8%9E%8D%E5%90%88%E6%95%88%E6%9E%9C.gif)(en-resource://database/629:1)
- 视频 [可以实现什么效果]

--- 

## 渲染管线
粗略地过一下每一个阶段里面做了什么，可以用来做什么。
![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/%E6%B8%B2%E6%9F%93%E7%AE%A1%E7%BA%BF%E6%B5%81%E7%A8%8B.jpg)
蓝色这次分享不关注。
绿色说明是可编程的阶段。
黄色是可配置的，无法完全自定义 ，但也可以修改暴露出来的参数。
紫色则是完全固定的。

### 应用阶段
>>平时开发之中使用的移动位置，IK动画，碰撞检测之类都处于这个。没有太多需要讲的地方。对之后的管线有影响的也会相应的改变之后使用的参数；比如对摄像机的操作。粗略地理解为在Unity普通开发之中对世界物体进行的操作也可以。
>> 
>>除此之外有个需要提到的点，就是应用阶段有对于不可视物体进行剔除。这和后面几何阶段说到的裁剪相同，这个一般是基于AABB包围盒级别的剔除

### 几何阶段
![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/%E5%87%A0%E4%BD%95%E9%98%B6%E6%AE%B5.jpg)
>>顶点着色器：常用在对顶点数据的处理，比如转换到后序流程需要的坐标系之中，然后传递给下一流程
>> 
>>曲面细分着色器：一般用于LOD。对于不同距离的物体采用不同精细度的表现。其内部还有更为详细的管线，感兴趣的可以自行了解。
>> 
>>几何着色器：这个着色器特点是可以对图元装配模式进行修改，接触过OpenGL的机会知道传入GPU的顶点可以有三种不同的装配模式，分别为点、线、三角形面。可以实现不少有趣的效果，比如毛发生成，模型爆炸后的粒子飞散效果。
>> 
>>投影，实际上只是一个准备阶段，并没有将3D投影在2D平面上。将视锥体转化为NDC。（Z-fighting）
>> 
>>裁剪，超出NDC之外的内容被裁剪掉。xyz: -1 ~ 1
>> 
>>屏幕映射，NDC中的内容被映射到当前分辨率的屏幕2D平面之中。

### 光栅化阶段
>> 
>> 三角形遍历，三角形生成。固定阶段，将顶点按照选择的装配模式组合起来，并且计算出包含在内部的片元（理解为又可能成为最终呈现在屏幕的像素前身即可）
>> 
>> 像素着色器，对每一个片元进行着色。在这个阶段之中每个像素采用的属性都通过重心坐标插值进行了平滑插值。
>> 
>> 逐片元操作，深度测试，透明度测试等，决定了片元最终是否作为像素呈现在屏幕之上。

--- 

## 如何实现引入时的效果
假设我们实现的是两个圆形的融合效果。那么只需要在一张Plane之中根据圆形的标准化方程来计算出每一个像素是否处于圆形之中。这个可以描述几何体边界的方程被称为SDF。

--- 

## SDF是什么
有向距离场（Signed Distance Function）
数学上，以原点为圆环，怎么描述
x² + y² - radius² = 0
带入一个圆外的点，则x² + y² - radius² > 0
带入一个圆内的点，则x² + y² - radius² < 0

同理可得 球形状的表达式为
x² + y² + z² - radius² = 0

其他几何体的SDF
[3D SDF](https://www.iquilezles.org/www/articles/distfunctions/distfunctions.htm)
[2D SDF](https://www.iquilezles.org/www/articles/distfunctions2d/distfunctions2d.htm)
这样，我们就可以绘制出单个几何体。

## 用SDF怎么描述两个物体之间的交互（几何布尔运算）
交max
```
function intersect(sdf1, sdf2, x,  y){
    reutrn  max(sdf1(x,y), sdf2(x,y))
}
```

并min 
```
function union(sdf1, sdf2, x,  y){
    reutrn  min(sdf1(x,y), sdf2(x,y))
}
```

差max(a,-b)
```
function substract(sdf1, sdf2, x,  y){
    reutrn  max(sdf1(x,y), -sdf2(x,y))
}
```

--- 

## 可实现效果：
![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/vid_00.gif)
- 视频 [怎么描述一只蜗牛]

--- 

## 效果移植到3D，如何实现
如果需要实现3D效果，比如两个球体进行交互，那么需要怎么实现出这一个效果。

--- 

## 光栅化思路
最开始的思路是通过对Unity之中原生的几何模型进行操作，Mesh修改以及颜色计算等。

### 光栅化流程：

> ![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/rasterization.gif)
> 
> 按照渲染管线的流程来理解，光栅化的流程就是将同一份材质之中的片元先像动图之中映射到屏幕空间的2D平面之中，再对每一个片元进行相应的计算。这也导致了对环境的低敏感度。

### 环境信息如何获取？

> 有一个例子就是：假设这个小球是金属材质。并且周边还有好几个不同颜色的小球。那么这个金属小球需要怎么处理才可以获取其他小球倒映在自身的效果呢？
> 
> 采用的一般是通过在自身位置采用多相机渲染，将相机的渲染结果作为反射的环境映射纹理。
> 
> 这样的操作本质上不是这个物品对周围环境进行了相应，而是在该位置上看到的环境再次贴上这个物品的表面来实现的Trick
> 
> 因此可以发现，如果不进行特殊处理，那么他渲染出来的效果往往也只是自身所收到的光照效果，无法获取到周围环境的变化。在光栅化的过程之中，每一个物体之中的像素都是仅关注自身所受到的直接影响

### 遇到的问题

> 但是两个物体产生互动时，他们各自的形变该如何计算，对形体以及颜色的改变又会有多少的贡献度来作为插值也没有头绪。


## 降维

> 为什么在2DPlane可以通过SDF实现？
> 
> 可以轻松的遍历2D平面的每一个像素，再判断每一个像素是否处于这个方程表达的范围之中，是的话则渲染。
> 
> 是判断3D世界之中的每一个像素？怎么获取所有的像素？

--- 

## 光线追踪 思路

> 如何获取三维世界所有像素？

### 光线追踪流程：

> ![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/RayTarce.gif)
> 
> 在视点对成像平面的每一个像素发射出一条射线，当射线与物体相交的时候，根据焦点的材质，对射线进行对应的反射、吸收等操作，最后决定出这个像素的颜色。
> 
> 可以发现，每一个2D像素的路径检测了空间之中对该2D像素有所贡献的所有3D空间像素。
> 
> 已有了起点，光线方向。则可以将该射线途径的点都进行判断

--- 

## Raymarching

> 不采用频繁而短的步长前进，而是将所有SDF封装为一个Map函数，传入当前点的位置，根据返回值，尽可能地最大限制的向前。当当前可前进的步长短到一定长度，则说明此时已于场景中物品进行了碰撞。
> 
> ![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/raymarching.jpg)

---

## Unity实现
借助了内置的Cube来作为光追的范围限制，可以减少计算的次数。在c#层之中将所有信息传递进shader之中，shader在片元着色器之中对每个像素进行Raymarching操作。
![](https://raw.githubusercontent.com/Tcyily/Tcyily.github.io/dev-MetaBall/_res/2021-12-26-RayMarching/SDF%E6%95%88%E6%9E%9C.gif)

### SDF应用
[Happy Jumping](https://www.shadertoy.com/view/3lsSzf)

[体积云源码](https://www.shadertoy.com/view/XslGRr)

http://www.phpheidong.com/blog/article/139785/1d9c27050ceb86b7e125/