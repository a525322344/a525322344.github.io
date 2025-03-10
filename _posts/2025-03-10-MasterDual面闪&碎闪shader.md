---
title: MasterDual面闪&碎闪shader
date: 2025-03-10
categories:
- CG
tags: 
- Unity
- Shader
---

​![image](/assets/images/2025-03-10-MasterDual/Unity_kt9yEMzwHT-20250309200047-59w6yz2.gif)​

不是完全还原，不过大体是那个意思了，并且为了看效果把扫光的频率速度调快了；

美术资源截帧获取的，纹理通道可以合并的处理了一下；

基本上都是是特效shader常规做法，涉及uv变换，颜色调整等，下面详细介绍一下。

### 面闪

#### 各个区域Mask贴图

​![image](/assets/images/2025-03-10-MasterDual/image-20250309210546-5x0j0kh.png)​

第一个Mask图R通道是边框遮罩，G通道是卡图遮罩，A通道是半透

第二张图是属性字遮罩图集；第三张图是卡名和星级遮罩，该卡是魔法卡，无星级。另外字体的RGB是法线，用于碎闪金字的效果。

CombineMask除了uv.y需要上下颠倒一下，直接采样

```hlsl
float2 uv = float2(i.uv.x, 1 - i.uv.y); 
float4 combineMaskMap = _CombineMask.Sample(sampler_BaseMap, uv);
```

NameStarMask需要调整一下uv，并且纹理需要设置成Clamp

```hlsl
float2 nameUV = float2(uv.x, uv.y * 5);
float4 nameStarMaskMap = _NameStarMask.Sample(sampler_NameStarMask, nameUV);
```

AttributeMask的uv需要先转化到对应位置，用此uv得到属性图标位置[0,1]的mask，再计算应用图集后的uv

```hlsl
float2 attriUV = uv;
attriUV += float2(-0.82, -0.048);
attriUV *= float2(9.0, 13.68);
float2 attriUVMask = step(0, attriUV) * step(attriUV, 1);
attriUVMask.x = attriUVMask.x * attriUVMask.y;

attriUV += _AttributeIndex;
attriUV *= 0.25;
float4 attributeMaskMap = _AttributeMask.Sample(sampler_BaseMap, attriUV);
```

​![image](/assets/images/2025-03-10-MasterDual/image-20250309215204-llrjat8.png)​

最后准备好的各区域遮罩如下

```hlsl
float edgeMask = combineMaskMap.r;
float faceMask = step(0.01,combineMaskMap.g);
float nameMask = nameStarMaskMap.a;
float attriMask = attributeMaskMap.r * attriUVMask.r;
```

#### 扫光

​![image](/assets/images/2025-03-10-MasterDual/image-20250309223245-xurdgfi.png)​

观察可知扫光的区域是卡框、卡名（星级）和属性，所以：

```hlsl
float swapMask = edgeMask + nameMask + attriMask;
```

​![image](/assets/images/2025-03-10-MasterDual/image-20250309221101-49cxddi.png)​

旋转uv，并且随时间游动采样渐变Mask

```hlsl
float2 rotateUV(float2 uv, float angle, float2 center = float2(0.5, 0.5))
{
    uv -= center;
    float cosA = cos(angle);
    float sinA = sin(angle);
    uv = float2(
        uv.x * cosA - uv.y * sinA,
        uv.x * sinA + uv.y * cosA
    );

    uv += center;
    return uv;
}

...
//SwapParams R:scale, G:angle, BA:speed
float2 swapUV = uv * _SwapParams.x;
swapUV = rotateUV(swapUV, _SwapParams.y);
swapUV += _Time.y * _SwapParams.zw;
float4 swapMaskMap = _SwapMask.Sample(sampler_BaseMap, swapUV);
float3 swapColor = swapMaskMap.rgb * swapMask;
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_xmojiPfRI6-20250309223904-8vsiazl.gif)​

#### 卡面闪烁

观察md的面闪，卡图实际上就是部分区域亮度的变化，没有虹彩的效果（碎闪有），所以使用3个通道的noise贴图，用不同方向随时间流动采样，结果相乘，可以让噪声更随机一些，得到的值经过数值调整，在略小于1和大于1之间变化，最终乘到卡图上，让卡面有一些明暗（更亮一些）变化

​![image](/assets/images/2025-03-10-MasterDual/image-20250309234308-c0n4e4d.png)​

```hlsl
//Face Swap Light
///FaceNoiseParams R:speed, G:scale
float2 faceUV = uv * _FaceNoiseParams.g;
float faceNoiseR = _FaceNoiseMap.Sample(sampler_BaseMap, faceUV + _Time.y * _FaceNoiseParams.r).r;
float faceNoiseG = _FaceNoiseMap.Sample(sampler_BaseMap, faceUV - _Time.y * _FaceNoiseParams.r).g;
float faceNoiseB = _FaceNoiseMap.Sample(sampler_BaseMap, faceUV + _Time.y * _FaceNoiseParams.r * float2(1, -1)).b;
float faceNoise = faceNoiseR * faceNoiseG * faceNoiseB * 20 + 1;

float timeLerp = sin(_Time.y * _FaceNoiseParams.r) * 0.5 + 0.5;
faceNoise = lerp(faceNoise, 1, timeLerp);
faceNoise *= faceMask;

float3 faceColor = basemap.rgb * faceNoise;
```

![image](/assets/images/2025-03-10-MasterDual/Unity_DUkFA2jlGi-20250309232246-l4w5est.gif)​

#### 整合

```hlsl
//Final
float3 finalColor = lerp(basemap.rgb, faceColor, faceMask) + swapColor;

return float4(finalColor, 1);
```

前面补上切圆角的clip

```hlsl
...
float4 combineMaskMap = _CombineMask.Sample(sampler_BaseMap, uv);
clip(combineMaskMap.a - 0.5);
...
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_Y8nsBJvVg6-20250309234416-g0r5xkm.gif)​

### 碎闪

开始前先定义一下Keywords，碎闪和面闪有些变量纹理是相同的，部分计算也可以共用，可以使用一个shader实现

```hlsl
[KeywordEnum(MDUR, MDSER)]_Rare("Rare", Float) = 0
...
SubShader
{
	Pass{
		...
		#pragma shader_feature_local _RARE_MDUR _RARE_MDSER
	}
}
```

#### 边框碎闪

​![image](/assets/images/2025-03-10-MasterDual/image-20250310132806-njtw3ca.png)​

只有edgeMask上有旋转的粉蓝白三色碎闪，形状有上面的三角形“法线”贴图提供；

通过旋转uv采样第一张“方向”贴图，和第二张法线贴图做类似漫反射计算的点乘，获取0-1的渐变值

```hlsl
//Flicker
float2 flickerDirUV = i.uv;
flickerDirUV = rotateUV(flickerDirUV, _Time.y * _FlickerSpeed);
float4 flickerDirMap = _FlickerDirMap.Sample(sampler_FlickerDirMap, flickerDirUV);
float3 flickerDir = flickerDirMap.xyz * 2 - 1;
float3 flickerNormal = _FlickerNormalMap.Sample(sampler_BaseMap, i.uv * 1.5).xyz * 2 - 1;
float flickerDot = dot(normalize(flickerDir), flickerNormal) * 0.5 + 0.5;
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_LiJWbvms1z-20250310141252-hn9zz70.gif)​

再用这个值去插值3个颜色，使用normal来处理一下明暗变化，然后应用到边框上；

```hlsl
float3 flickerColor = lerp(_FlickerColor1.rgb, _FlickerColor2.rgb, saturate(flickerDot * 2));
flickerColor = lerp(flickerColor, _FlickerColor3.rgb, saturate(flickerDot * 2 - 1));
flickerColor *= (flickerNormalMap.r + flickerNormalMap.g) * 0.5;
flickerColor *= edgeMask;//仅作展示
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_s3HuKwqvmz-20250310142215-2abfg8m.gif)​

#### 卡名金字

卡名颇有一些金属质感，这里用简单的光照模型，并且使用上一部的碎闪模拟全局反射，来体现金字质感

```hlsl
//Gold name
float3 nameNormal = normalize(nameStarMaskMap.xyz * 2 - 1);
float3 fakeLightDir = normalize(_FakeLightDir);
float3 diffuse = dot(nameNormal, fakeLightDir) * 0.25 + 0.75;
diffuse *= _TextColor.rgb;

float3 halfDir = normalize(fakeLightDir + float3(-0.5, 0.5, 1));
float3 specular = dot(nameNormal, halfDir);
specular = saturate(specular * specular);
specular *= flickerColor;

float3 nameColor = diffuse + specular;
nameColor *= step(0.15, nameMask);//仅作展示
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_kThcBM4tWJ-20250310151112-456t2me.gif)​

#### 卡面虹彩

复用面闪卡面计算noise的部分，叠乘FlickerDirMap.rgb作为虹彩颜色

```hlsl
//Face rainbow
float3 rainbow = flickerDirMap.rgb * 0.5 + 0.5;
rainbow *= (faceNoiseR + faceNoiseG + faceNoiseB) * 2 + 0.5;
rainbow = lerp(rainbow, 1, timeLerp) * basemap.rgb;
rainbow *= faceMask;//仅作展示
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_Qky8CNRsmJ-20250310152637-t2gkj0w.gif)​

#### 旋转光

​![image](/assets/images/2025-03-10-MasterDual/image-20250310153721-bph47hg.png)​

作用于整个卡的旋转的光效，并且把前面的效果整合

```hlsl
//Rotate Light
float4 rotateMap1 = _RotateLightMap.Sample(sampler_FlickerDirMap, flickerDirUV);
float2 rotateMapUV = rotateUV(flickerDirUV, -_Time.y * _FlickerSpeed);
float4 rotateMap2 = _RotateLightMap.Sample(sampler_FlickerDirMap, rotateMapUV);
float rotateLight1 = rotateMap1.g - rotateMap1.g * rotateMap1.a;
float rotateLight2 = rotateMap2.g - rotateMap2.g * rotateMap2.a;
rotateLight1 += rotateLight2;
rotateLight1 = rotateLight1 * 0.25 + 0.75;

//Final
float3 faceColor = basemap.rgb * faceNoise;
finalColor = lerp(finalColor, nameColor * 1.5, step(0.15, nameMask));
finalColor = lerp(finalColor, rainbow, faceMask) + flickerColor * edgeMask;
finalColor *= rotateLight1;
```

![image](/assets/images/2025-03-10-MasterDual/Unity_LUkuXgIKet-20250310160154-geqywir.gif)​

#### 流动碎闪

​![image](/assets/images/2025-03-10-MasterDual/image-20250310154943-zyzp8ek.png)​

最后是流动的碎闪膜，会用到之前的FlickerNormal和一个随时间变化的随机方向做点乘，再用这个点乘结果作为uv采样上面的贴图，模拟虹彩颜色

```hlsl
//Rain Swap
float v = _Time.y * _SwapParams.b;
float3 randomDir = normalize(float3(sin(v), sin(v + 1), sin(v + 2)));
float rainSwapDot = dot(randomDir, flickerNormal);
rainSwapDot *= swapMaskMap.r;
float3 rainSwapColor = _RainbowMap.Sample(sampler_BaseMap, rainSwapDot.rr).rgb;
rainSwapColor *= swapMaskMap.r * _RainbowIntensity;

...
finalColor += rainSwapColor;
```

​![image](/assets/images/2025-03-10-MasterDual/Unity_4nzO6GDAs9-20250310160552-7fq4h7j.gif)​
