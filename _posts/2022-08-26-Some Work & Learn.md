# Work

## Render

### ShapeFX shader

Houdini 特殊处理模型 uv 通道数据，实现的顶点动画效果

![fx.gif](/assets/images/2022-08-26-WorkLearn/fx-20220112145448-hch6a40.gif)

### GFX ShapeFX shader

GFX 特效 shader 中添加 ShapeFx 功能

![录制_2022_01_13_11_36_05_861.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_11_36_05_861-20220113130631-h0vm8p7.gif)

### Distort shader

抓取屏幕进行处理的着色器，包含扰动、颜色分离、多种模式模糊效果等。

![录制_2022_01_13_11_31_16_513.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_11_31_16_513-20220113130629-y7tfulz.gif)![录制_2022_01_14_16_44_31_725.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_14_16_44_31_725-20220114164722-v4e1ync.gif)

#### GFX ShapeFx 和 Distort 结合的效果

![录制_2022_01_13_11_00_53_30.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_11_00_53_30-20220113130627-xee7ru5.gif)

### Decal shader 视察深度效果

![录制_2022_01_13_13_51_01_107.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_13_51_01_107-20220113135119-cdidokd.gif)

### 全息效果 shader

使用一个 pass 完成，破碎、边缘抖动、碰撞挤压效果基于顶点位移，着色基于 PBR 框架完成。

![录制_2022_01_14_16_19_32_149.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_14_16_19_32_149-20220114162116-kay5q50.gif)

### PBR 卡通 NPC shader

效率优先、单个材质完成角色身体、面部、眼睛、头发（低级的 npc 倾向于没有头发）的渲染

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220114164712-wko0skv.png)

### textMeshPro 组件 故障字体 shader

![录制_2022_01_12_14_09_15_47.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_12_14_09_15_47-20220112145450-x5bw3lq.gif)

### 粒子特效用简谐顶点动画 shader

![录制_2022_01_14_16_23_37_990.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_14_16_23_37_990-20220114162408-nlxni29.gif)

### fake interior 假窗口效果

![录制_2022_01_13_15_01_21_221.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_15_01_21_221-20220113150219-83514ms.gif)

### Max DX shader 

规则和 Unity 项目类似，用于美术直接在 max 中预览涉及顶点色、uv 通道（头发的渐变、眼睛与面部的插值、描边粗细）的效果

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220114172654-2v6080b.png)

### 基于模板测试的外描边 shader

![录制_2022_01_14_17_07_50_723.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_14_17_07_50_723-20220114170810-6o7j033.gif)


## Editor

### Unreal-Max LiveLink动画编辑工具

![录制_2022_08_26_18_26_05_539](/assets/images/2022-08-26-WorkLearn/录制_2022_08_26_18_26_05_539-20220826182713-z4ke208.gif)

### Unreal LookDev发布版

![saveload](/assets/images/2022-08-26-WorkLearn/saveload-20220826182041-bu2yh3m.gif)

### Shader Debug 编辑器&运行时工具

![录制_2022_08_26_17_18_08_0](/assets/images/2022-08-26-WorkLearn/录制_2022_08_26_17_18_08_0-20220826172110-kns08s6.gif)

![录制_2022_08_26_17_20_25_895](/assets/images/2022-08-26-WorkLearn/录制_2022_08_26_17_20_25_895-20220826172115-1g6j784.gif)

### TimeLine Camera 可视化贝塞尔曲线工具

![录制_2022_08_26_14_55_14_52](/assets/images/2022-08-26-WorkLearn/录制_2022_08_26_14_55_14_52-20220826171215-umjisrj.gif)

### Houdini-Unity远程烘培工具

![录制_2022_08_26_17_11_13_556](/assets/images/2022-08-26-WorkLearn/录制_2022_08_26_17_11_13_556-20220826171144-552ves2.gif)

### 角色prefab 制作工具

一键制作 prefab 资产，可直接进包；根据路径匹配类型，进行相应的操作；更迭 fbx 文件，重新制作可继承各种旧 prefab 数据；一个 fbx 可对应多个 prefab，以 json 文件记录。

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113135221-kzlwi7t.png)

### 场景 prefab 制作工具

增加自动正确赋予多维子材质顺序的功能，根据 max 工具导出的 json 文件进行判断。

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113135733-ga75cm5.png)

### prefab 检查工具

通过 debug 形式进行输出

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140026-iie5y0u.png)

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140032-mo6yspt.png)

### max 生成动作实际位移节点、及塌陷动作到原地工具

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140359-8y73f6g.png)

### 角色 fbx 输出工具

增加白名单功能，通过 json 文件，过滤可以输出的对象

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140413-oogg635.png)

### 转换骨骼工具

自动将骨骼在 bip 类型 bone 类型之间转换，以供动作同事修改骨骼动画

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140427-9sna4hp.png)

### 场景 fbx 输出工具

处理多维子 mesh，使其材质 id 显示顺序与 导入 Unity 中 fbx 的子材质顺序一致，并输出记录子材质信息的 json文件

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140437-kb9zs9p.png)

### fbx 缩放工具

批量修改 fbx 文件的缩放大小

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113140531-x1kekox.png)

### 骨骼驱动布料模拟工具

程序化进行布料模拟的准备工作

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220113141141-t5da5ix.png)

# Learn

## Render

### GAMES202学习

#### Shadow

* Shadow Mapping

  ![image.png](/assets/images/2022-08-26-WorkLearn/image-20220525205349-f8nkfhl.png)
* PCF

  ![image.png](/assets/images/2022-08-26-WorkLearn/image-20220526115529-cgwi6cd.png)
* PCSS

  ![image.png](/assets/images/2022-08-26-WorkLearn/image-20220526161502-7lb6ljz.png)

#### Diffuse PRT

![录制_2022_06_30_23_55_28_76.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_06_30_23_55_28_76-20220704202900-m7zlz5d.gif)

#### Screen Space Ray Traching

* 只有直接光照

  ![image.png](/assets/images/2022-08-26-WorkLearn/image-20220710184407-dmuq079.png)
* SSR

  ![image.png](/assets/images/2022-08-26-WorkLearn/image-20220712220036-2whf1wx.png)

#### Kulla-Conty BRDF

通过预计算生成贴图，实时采样计算，弥补BRDF(下)因忽略了平面之间光线多次弹射而损失的能量，结果为上图

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220718233500-eq5b65t.png)

#### Real Time Ray Trach 降噪

* 输入有这些缓冲等：

  ![image](/assets/images/2022-08-26-WorkLearn/image-20220731160054-lnd174s.png)

  ![image](/assets/images/2022-08-26-WorkLearn/image-20220731160210-1he4bcc.png)

  ![image](/assets/images/2022-08-26-WorkLearn/image-20220731160242-qgm5gjj.png)
* 频域的联合双边滤波

  ![image](/assets/images/2022-08-26-WorkLearn/image-20220731155837-reajiot.png)
* 时域上多帧混合

  ![image](/assets/images/2022-08-26-WorkLearn/image-20220731155954-uu5mlf9.png)

### 视察 shader

![录制_2022_01_13_14_38_09_279.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_14_38_09_279-20220113144628-29rg4ty.gif)

### PBR shader

![image.png](/assets/images/2022-08-26-WorkLearn/image-20220114175613-2om1k3g.png)

### 程序纹理

![录制_2022_01_13_14_57_10_901.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_14_57_10_901-20220113145734-wx6vp2e.gif)

### 虹彩效果的卡牌

![录制_2022_01_13_14_59_01_837.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_14_59_01_837-20220113145940-8f45166.gif)

### 闪闪发光的方块 shader

![录制_2022_01_13_15_07_13_44.gif](/assets/images/2022-08-26-WorkLearn/录制_2022_01_13_15_07_13_44-20220113150814-u8a85g8.gif)

## Editor

### 噪声生成工具

基于Compute Shader的noise生成工具，能够自动识别特定目录下的Compute shader，自动获取其中的Kernel和属性，并显示在UI面板上。

![录制_2022_08_26_20_27_41_768](/assets/images/2022-08-26-WorkLearn/录制_2022_08_26_20_27_41_768-20220826202952-d83mu3n.gif)
