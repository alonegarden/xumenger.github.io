---
layout: post
title: 从零开始做游戏：Mixamo绑定动画导入Unity
categories: 游戏开发之blender 游戏开发之unity 
tags: 游戏 建模 blender 贴图 3D 雕刻 绑定骨骼 动画 快捷键 摸索Blender 骨骼 骨骼权重 Rigging:Rigify unity unity3d Unity3D U3D 镜像模式 fbx mixamo 从零开始做游戏
---

## Blender造人



## Mixamo绑定动画



## Unity中应用模型动画

为这个模型添加一个C# 脚本，脚本内容如下

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CuberScript : MonoBehaviour
{
    // 控制动画
    private Animator anim;

    // 控制摄像头随角色移动
    private Vector3 offset = Vector3.zero;
    private Camera mainCamera;


    void Start()
    {
        anim = GetComponent<Animator>();

        // 初识时获取模型和摄像机的偏移值
        mainCamera = Camera.main;
        offset = mainCamera.transform.position - transform.position;
    }

    
    void Update()
    {
        /**
         * 控制动画
         */
        float vertical = Input.GetAxis("Vertical");                      // 垂直轴（w、s或者上下键）
        float horizontal = Input.GetAxis("Horizontal");                  // 水平轴（a、d或者左右键）
        Vector3 dir = new Vector3(horizontal, 0, vertical);              // 方向向量

        if (dir != Vector3.zero)
        {
            transform.rotation = Quaternion.LookRotation(dir);           // 旋转
            transform.Translate(Vector3.forward * 6 * Time.deltaTime);   // 移动
            anim.SetBool("walk", true);                                  // 播放行走动画
        }
        else
        {
            anim.SetBool("walk", false);                                 // 播放站立动画
        }


        /**
         * 摄像头跟随人物
         */
        mainCamera.transform.position = transform.position + offset;
    }

}
```

>详细的操作过程可以参见我对应录制并上传到[bilibili 的视频教程]()！

## 参考资料

* [[傅老師/Blender教學] 03 - 做個方塊人(Make a Blockman)](https://www.bilibili.com/video/av16721533)
* [[傅老師/Blender教學] 04 - 使用第三方服務綁骨(Rigging with Mixamo)](https://www.bilibili.com/video/av16738239)
* [摸索Blender：Blender常用基础操作](http://www.xumenger.com/blender-example-01-20190907/)