<!--
 * @Author: your name
 * @Date: 2022-03-01 00:45:41
 * @LastEditTime: 2022-03-01 01:21:14
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \tcyily.github.io\_posts\2022-2-28-RoomScene1-Fabric.md
-->
# 樱花兔-项目RoomScene学习-地毯效果

## 1. 项目技术栈
    采用了HRDP + ShaderGraph来实现

--- 

## 2. 个人复刻
    1. 采用URP
    2. 同样采用ShaderGraph
    3. 如何提高原项目的帧率（60FPS in 3070）

--- 

## 3. 实现思路
    采用

---

## 4. 关键点  

### 4.1. ShaderGraph节点 - Parallax Occlusion Mapping  
    问题1：该节点有什么作用 
        译为视差遮蔽映射；在项目之中将视角移至与地毯平齐，发现并没有顶点生成绒毛。推测功能为类似法线贴图，在不改变模型的情况下通过光线的作用来模拟物体表面的起伏  

    问题2.内部是如何实现的
        需要：高度图x1，法线贴图x1，漫反射颜色映射贴图x1
        步骤：
            1. 0.0水平面为Plane的初始高度
            2. 对T0的地方进行高度图的采样，发现深度为0.55;
            3. 那么视线V还可以进行往前走，直到T1点
            4. 使用T1点的位置作为UV，对颜色贴图，法线贴图采样
>![](../_res/2022-2-28-RoomScene1-Fabric/Theory.jfif)  

是不是很像体渲染？原有的Plane相当于一个成像平面。在于该平面接触之后，判断是否于体素有相交，没有则进行往前步进。  

> [资料1： [译] GLSL 中的视差遮蔽映射（Parallax Occlusion Mapping in GLSL](https://segmentfault.com/a/1190000003920502)


