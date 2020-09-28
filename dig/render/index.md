---
title: 计算机图形学
layout: page
comments: no
---

>[第 N-1 手资料](http://www.xumenger.com/docs-20200916/)

>[https://github.com/xumenger/UnityToonShader](https://github.com/xumenger/UnityToonShader)

>[https://github.com/xumenger/Unity_Shaders_Book](https://github.com/xumenger/Unity_Shaders_Book)

>[https://github.com/xumenger/Awesome-Unity-Shader](https://github.com/xumenger/Awesome-Unity-Shader)

>[https://blog.csdn.net/zhmxy555/category_9264739.html](https://blog.csdn.net/zhmxy555/category_9264739.html)

## 文章目录

* [3D图形学理论基础](#3D图形学理论基础)
* [Unity Shader 基础结构](#Unity Shader 基础结构)
* [Unity Shader调试](#Unity Shader调试)

## 3D图形学理论基础




## Unity Shader 基础结构

比如下面是编写Unity Shader 的基本框架！

```shader
Shader "Test/ShaderExample"{

    Properties {
        // 属性
        // Name ("display name", PropertyType) = DefaultValue

        // Numbers and Sliders
        _Int ("Int", Int) = 2
        _Float ("Float", Float) = 1.25
        _Range ("Range", Range(0.0, 5.0)) = 3.0

        // Colors and Vectors
        _Color ("Color", Color) = (1, 1, 1, 1)
        _Vector ("Vector", Vector) = (2, 3, 6, 1)

        // Texture
        _2D ("2D", 2D) = "" {}
        _Cube ("Cube", Cube) = "white" {}
        _3D ("3D", 3D = "black" {}
    }

    SubShader {
        // 显卡A使用的子着色器

        // 标签，可选的
        [Tags]

        // 状态，可选的
        [RenderSetup]

        // 每个Pass 定义了一次完整的渲染流程，但如果Pass 数量过多，会造成渲染性能的下降
        Pass {
            [Name]
            [Tags]
            [RenderSetup]

            // Other Code

        }

        // Other Pass

    }

    SubShader {
        // 显卡B使用的子着色器

    }

    // 如果显卡对于上面的子着色器都不支持的话，会使用Fallback 语义指定的Unity Shader
    Fallback "VertexLit"
}
```

## Unity Shader调试

在Unity 的Project 窗口选中Unity Shader 文件后，对应在Inspector 窗口点击【Compile and show code】下拉列表可以让开发者检查该Unity Shader 针对不同的图形编程接口（例如OpenGL、D3D9 等）最终编译生成的Shader 代码，可以利用这些代码来分析和优化着色器！