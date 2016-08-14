---
layout:     post
title:      "Image Effect之Vignette"
subtitle:   ""
date:       2016-08-14
author: 	"YW"
header-img: "img/vignette.jpg"
tags:
    - Shader
    - Image Effect
---

### 原理
vignette算是ImageEffect里非常简单也是非常常用的效果之一，实现的算法也非常简单：

	1.算出采样点的uv坐标距离屏幕中心的位置d
	2.如果uv坐标超出了中心圆（或椭圆）的半径r，那么就是黑色，如果小于半径r，就是亮的
	
在计算uv坐标距离屏幕中心的位置之前，需要把uv坐标的原点平移到屏幕中心，为了方便计算，同时把UV坐标映射到[-1, 1]的区间，这样的话亮的区域就是距离原点的距离小于1的区域。
  
    float2 uv =( i.uv - 0.5) * 2;// 将uv坐标转换到[-1, 1]区间，原点是屏幕中间
	
在计算uv坐标到屏幕中心位置时，没有必要使用开方，因为大于1的数的平方仍然大于1，小于1的数的平方仍然小于1

    float uvDot = dot(uv, uv);   
	
### 实现
	Shader "Hidden/YW/Vignette"
	{
        Properties
	    {
            _MainTex ("Texture", 2D) = "white" {}
            _VignetteInstensity ("vignetteIntensity", Range(0, 10)) = 1
            [Toggle(SMOOTH)] _Smooth ("Smooth", int) = 1
        }
        SubShader
        {
            Cull Off ZWrite Off ZTest Always
    
            Pass
            {
                CGPROGRAM
                #pragma vertex vert_img
                #pragma fragment frag
                #pragma multi_compile __ SMOOTH
                
                #include "UnityCG.cginc"
                
                sampler2D _MainTex;
                float _VignetteInstensity;
                
                fixed4 frag (v2f_img i) : SV_Target
                {
                	float2 uv =( i.uv - 0.5) * 2;// 将uv坐标转换到[-1, 1]区间，原点是屏幕中间
                	//float uvDot = sqrt(dot(uv, uv));// 计算转换后的uv坐标到屏幕中间的距离		
                	float uvDot = dot(uv, uv); // 因为是半径为1的圆，小于1的数乘以小于1的数还是小于1，所以使用距离的平方也能近似的得到结果
                	uvDot = uvDot * _VignetteInstensity; // 乘以强度，强度越大黑色区域越大
                	#ifndef SMOOTH
                	uvDot = step(1, uvDot);		
                	#endif
                	float mask = saturate(1 - uvDot); 
                
                	fixed4 col = tex2D(_MainTex, i.uv);
                	return col * mask;
                }
                ENDCG
            }
        }
    }	
    
## 效果
边缘硬化效果
![alt text](/img/vignette1.jpg)
边缘柔和效果
![alt text](/img/vignette2.jpg)