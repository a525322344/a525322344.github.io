---
title: Compute Shader初步理解
date: 2022-08-07
categories:
- CG
tags: 
- Unity
---

在unity中，新建一个compute shader，它的内容会是这个样子的：

```c
// Each #kernel tells which function to compile; you can have many kernels
#pragma kernel CSMain

// Create a RenderTexture with enableRandomWrite flag and set it
// with cs.SetTexture
RWTexture2D<float4> Result;

[numthreads(8,8,1)]
void CSMain (uint3 id : SV_DispatchThreadID)
{
    // TODO: insert actual code here!

    Result[id.xy] = float4(id.x & id.y, (id.x & 15)/15.0, (id.y & 15)/15.0, 0.0);
}
```

这些关键字都起什么作用？

先看这张图，总的来说，Dispatch包含了许多线程组，每一个线程组包含了许多线程，而这一堆线程并行的运行在GPU上。

![image.png](/assets/images/2022-08-07-ComputeShader/image-20220707113116-q3bpiff.png)

* Dispatch，即上面的三维表格，可以叫它“线程组们”？总之是三维结构存储的线程组的表，每一个格子是一个线程组，Dispatch(5,3,2)表示其中有5 * 3 * 2 = 30个线程组。

* SV_GroupID是当前线程组在Dispatch中的位置，是一个三维变量。

再看线程组，也就是下面的三维表格，每一个格子是一个线程，unity compute shader 中的[numthreads(8,8,1)]描述的就是线程组的大小，即8 * 8 * 1 = 64 个。

* SV_GroupThreadID是当前线程在线程组中的位置，是一个三维变量。

* SV_DispatchThreadID是当前线程在Dispatch中的位置，computeshader中的位置就是这个，此坐标综合考虑了在Dispatch中，所有线程的次序，是跨线程组的线程坐标位置。

![image](/assets/images/2022-08-07-ComputeShader/image-20220804182912-fdhwqa4.png)

或许一时不能理解为什么可以这样算，得到的结果为什么可以当作"总的坐标“?

不妨让z轴为1，以二维的Dispatch(4,4,1)和二维(3,3,1)的线程组推算一下。

![image](/assets/images/2022-08-07-ComputeShader/net-img-image-20220804170015-jliia3s-20220804222758-d62k1ks.png)

SV_GroupID = (2,1,0) 

SV_GroupThreadID = (0,2,0)

在整个Dispatch中线程的坐标:

x = SV_GroupID.x * [线程组x轴长度]  + SV_GroupThreadID.x  
   = 2*3 + 0  
   = 6

y = SV_GroupID.y * [线程组y轴长度]  + SV_GroupThreadID.y  
   = 1*3 + 2  
   = 5

z = 0

整体即：  
SV_DispatchThreadID = SV_GroupID * [3,3,1] + SV_GroupThreadID


回过头来看unity生成的compute shader:

`[numthreads(8,8,1)]` 是一个线程组的大小

`uint3 id : SV_DispatchThreadID` 是当先线程在Dispatch上的位置

其他的代码则与c#如何运行compute shader有关

`#pragma kernel CSMain` 定义了一个shader程序入口，类似定义vert或frag，只不过这个程序需要c\# 脚本来调用执行

`RWTexture2D<float4> Result;` shader计算的结果储存在其中


下面是一个c\# 执行compute shader来生成图片的简单例子

compute shader:

```glsl
#pragma kernel CSMain

//图片的长度，需要从外部c#代码传入
float TextureSize;
//储存结果的数据结构不是唯一的，这里每一个线程只需要保存一个float值，我们使用float[]
RWStructuredBuffer<float> Result;

[numthreads(8,8,1)]
void CSMain (uint3 id : SV_DispatchThreadID)
{
    int index = id.x + TextureSize * id.y;

    Result[index] = (id.x + id.y)/TextureSize;
}
```

c\# :

```csharp
int texSize = 512;
//用来保存从compute shader中获取的结果
var resultArr = new float[texSize * texSize];
//创建computer buffer,第二个参量是compute shader中缓冲数据一个元素的字节大小，这里float的是4
var resultBuffer = new ComputeBuffer(resultArr.Length, 4);
//传入TextureSize
computeShader.SetFloat("TextureSize", texSize);
//这里计算的是Dispatch中线程组的长和宽
//我们要生成texSize*texSize的贴图，而compute shader（即线程组）的大小是[numthreads(8,8,1)]的
//所以Dispatch的长和宽为texSize/8
int threadGroup = Mathf.CeilToInt(texSize / 8.0f);
//获取名称为”CSMain“的kernel的序号，我们的shader中只有一个kernel，这里获取的只会是0
int kernel = computeShader.FindKernel("CSMain");
//绑定compute shader中的”Result“与外部定义的buffer
computeShader.SetBuffer(kernel, "Result", resultBuffer);
//执行compute shader，这里用到的变量上面都介绍了
computeShader.Dispatch(kernel, threadGroup, threadGroup, 1);
//从buffer中获取数据到resultArr
resultBuffer.GetData(resultArr);
//释放buffer内存
resultBuffer.Release();

//这里是把float[]生成为texture2D的代码
var tex = NoiseHelper.ConvertToSingleColorTex(texSize, resultArr);
NoiseHelper.SaveTextureToPNG(tex, savePath, texName);
```

结果

![image](/assets/images/2022-08-07-ComputeShader/image-20220804191312-qos2dts.png)
