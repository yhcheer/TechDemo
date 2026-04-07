# TechDemo

此页面所有素材是本人从零碎的工作笔记中整理，仅本人找工作使用，请勿外传和转载。

![image-20260407180710305](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180710305.png?raw=true)

![image-20260407180722481](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180722481.png?raw=true)

### Real Time Shadow Render Pipeline

- pre calcucate pass(bake static shadow & static object only)
- shadow cullings(also culling static object)
- prez  pass
- shadow cast pass(CSM dir light shadow & local light shadow & per character shadow)
- contact shadow pass(ray marching)
- screen space shadow pass(soft shadow & static shadow decode)
- shadow receive(sample screen space shadow texture)

### 稀疏阴影树 SparseShadowTree（SST）

普通的CSM在面对大世界场景中，也会有点压力，尤其是现在的手机平台上。这使得我们必须对以前的阴影算法进行改进。经调研，Treyarch在顶会SIGGRAPH 2016 TALK分享了其在《使命召唤：黑色行动3》的大型室外场景中应用了SST技术，取得不错的效果。

稀疏阴影树本质是一种静态阴影压缩方案，可以和常规的动态阴影算法（如CSM）融合作为室外大世界场景的方向光阴影解决方案。该算法的优势是降低开销的同时，还能保证高质量阴影的高精度。

![image-20260407180830763](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180830763.png?raw=true)

### 阴影烘焙与压缩

![image-20260407180855215](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180855215.png?raw=true)

![image-20260407180955254](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180955254.png?raw=true)

### 实时稀疏阴影树 RealTime SparseShadowTree（RSST）

我们发明了GPU实时阴影压缩算法RSST，耗时只需要related work的58%，同时保持误差精度在1e-4以内。我们也是业内第一个把该算法落地到移动端上的。

对比传统CSM方案，我们动态静态阴影分离，实时烘焙、压缩并缓存静态阴影，降低dp，最终GPU耗时减少50%，culling耗时减少70%，帧数提升67%。

对比竞品CSM Scrolling方案，culling频率更低，静态物体绘制次数更少。culling阶段总耗时降低25%，dp数下降6%，帧数提升15%。

RSST方案不仅优化了高dp下的CPU开销，并且优化了高面数下的GPU开销。对比第二章的相关工作章节，RSST对比Shadow Cache有更好的内存优势。对比传统SST，有灵活的动态光源的支持。对比CSM Scrolling，有更少的CPU culling开销。

![image-20260407182827426](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407182827426.png?raw=true)

![image-20260407183734210](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407183734210.png?raw=true)

![image-20260407183127828](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407183127828.png?raw=true)

### Cascade Shadow Map

Shadow Frustum优化，使用包围球构造，更紧凑更stable。比如光源旋转的时候，AABB的frustum会频繁更改size，导致抖动。

![image-20260407180448264](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180233347.png?raw=true)

级联过渡

![image-20260407170224503](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407170224503.png?raw=true)

距离淡出

![image-20260407170707126](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407170707126.png?raw=true)

### 点光阴影

实现了两种方案：立方体 Cube 和 双曲面 DPSM。

裁剪方案 cull by light ，通过构造AABB做剔除。

point shadow 40盏灯测试。立方体算法170FPS, 双曲面算法440FPS。

DPSM可进一步做平面分割优化。

![image-20260407163551378](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407163551378.png?raw=true)

### 聚光灯阴影

![image-20260407163615107](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407163615107.png?raw=true)



### 角色阴影

使用自定义bounding frustum渲染per character shadowmap，最大化利用shadowmap的精度，减少因精度不够带来的走样和抖动。

![image-20260407190356964](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407190356964.png?raw=true)

### 软阴影

PCF

​	使用的unity的tent滤波方案，提高fliter范围增加的耗时可以接受

![image-20260407175434809](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407175434809.png?raw=true)

![image-20260407164248310](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407164248310.png?raw=true)

PCSS

​	使用Poisson Disc Sampling

![image-20260407164115043](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407164115043.png?raw=true)

VSM和ESM

![image-20260407174519374](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407174519374.png?raw=true)





### 屏幕空间阴影

减少overdraw，方便结合其他阴影技术，提高贴图命中率。

![image-20260407164309730](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407164309730.png?raw=true)



### 半影mask

软阴影的采样数较大，通过做半影mask，仅在半影区域采样软阴影可以有效提高帧数。

![image-20260407164408508](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407164408508.png?raw=true)

430fps提高到520fps，帧数提高20%。

![image-20260407164823765](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407164823765.png?raw=true)



### 接触阴影

通过ray marching从shading point往光源发射射线并步进，查询是否被遮挡，有效修复因bias产生的peter panning现象。

![image-20260407171008575](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407171008575.png?raw=true)

还能为植被提供阴影。

![image-20260407172242750](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407172242750.png?raw=true)

还能为砂石地面提供细节阴影。还能为高精度模型提供很好的细节阴影。

![image-20260407172127333](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407172127333.png?raw=true)



### 半透明阴影

![image-20260407172635035](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407172635035.png?raw=true)



### 阴影与不同材质

![image-20260407173005372](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407173005372.png?raw=true)

### Shadow Proxy

shadow caster lod

一键生成简模，并在模型身上挂一个proxy模型，proxy模型不参与render只参与投影，有效降低dp和prim数。如果没有算法专门生成proxy的话，可以用低级lod的画阴影。

减面比例设置为30%，prim降低87w(51%)，帧数提升20%。移动端iphone11下，CSM pass耗时5.43ms降低为1.58ms。

![image-20260407181311328](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407181311328.png?raw=true)

### 镜子与阴影

旧管线的Mirror无法产生阴影。新管线的多View渲染支持的比较好。Mirror相当一个新的View，有自己的shadow cast。

![image-20260407173305299](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407173305299.png?raw=true)



### 一些项目组定制的魔改需求

实际生产环境中，因为服务多个项目组，经常会有些不物理，甚至奇怪的定制化需求。

逐光源 shadow color，修改shadow最终的结果，不再返回一个通道，而是叠加自定义的color。

![image-20260407173414880](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407173414880.png?raw=true)

自定义非方向光的阴影方向

![image-20260407180618923](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407180618923.png?raw=true)

ShadowOnly（ShadowProxy）

![image-20260407184138248](https://github.com/yhcheer/TechDemo/blob/main/images/image-20260407184138248.png?raw=true)