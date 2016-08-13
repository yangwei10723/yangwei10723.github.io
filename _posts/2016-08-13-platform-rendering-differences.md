---
layout:     post
title:      "不同平台渲染的差异"
subtitle:   ""
date:       2016-08-13
author: 	"YW"
header-img: "img/view.jpg"
tags:
    - Shader
---


##Render Texture coordinates

在Direct3D类和OpenGL类的的平台，贴图的竖直方向的坐标是不一样的
	1）在Direct3D类，Metal和主机上Y坐标轴的原点在左上，
	2）在OpenGL和OpenGL ES平台，Y坐标轴的原点在左下，

在大部分情况下我们都不需要去关注这个问题，如果在非OpenGL类的平台，使用了相机的Rendering into Render Texture时，Unity内部会将Render Texture反转。而另外两种情况需要我们自己来反转，①制作Image Effects ②在UV空间做渲染。

1.ImageEffects,颠倒的的RenderTexture，MSAA(多重采样抗锯齿)
在同时使用了ImageEffects和抗锯齿时，Unity的“非OpenGL平台自动反转Render into Texture"不会生效。在这种情况下Unity会先显然到屏幕上进行抗锯齿处理，然后渲染到RenderTexture进行ImageEffect效果处理，此时的ImageEffect的Source Image是没有进行上下反转的。
	如果是个简单的ImageEffect(一次只处理一张Texture)，那么Graphics.Blic函数会处理SourceRenderTexture没有上下反转的问题。如果在ImageEffect中处理多张RenderTexture，则需要自己对RenderTexture进行反转，代码如下：

	#if UNITY_UV_START_AT_TOP
		if(_MainTex_TexelSize.y < 0)
			uv.y = 1- uv.y;
	#endif

在使用GrabPss时也需要自己处理反转问题，而且在shader中对GrabPass贴图采样应该使用UnityCG文件中的 ComputeGrabScreenPos。

2.渲染纹理坐标(UV)
通过内置变量 _ProjectionParams.x可以判断出投影是否上下反转，+1表示已经反转，-1表示未反转，参考代码如下：
	
	float4 vert(float2 uv : TEXCOORD0) : SV_POSITION
	{
		float4 pos;
		pos.xy = uv;
		if (_ProjectionParams.x < 0)
		{
			pos.y = 1 - pos.y;
		}
		pos.y = 0;
		pos.w = 1;
		return pos;
	}


裁剪空间坐标系的不同
	1）D3D，Metal和主机平台，裁剪空间的近裁剪面的深度值是0，远裁剪面的深度值是1
	2）OpenGL类平台的，裁剪空间的近裁剪面的深度值是-1，远裁剪面的深度值是+1
	在shader中可以使用UNITY_NEAR_CLIP_VALUE宏来获取不同平台上近裁剪面的深度值。
	在脚本中可以用GL.GetGPUProjectionMatrix将Unity的坐标系转换到目标平台。


Shader计算的精度	
	对于PC上的GPU来说，所有的浮点数(float, half, fixed)都是一样的，使用32位精度来计算。而移动平台的GPU是不一样的。


在Shader中声明常量
	在HLSL中常量可以在任何地方初始化，而在GLSL中常量必须在编译时初始化。
	

>原文地址: [http://docs.unity3d.com/Manual/SL-PlatformDifferences.html](http://docs.unity3d.com/Manual/SL-PlatformDifferences.html)