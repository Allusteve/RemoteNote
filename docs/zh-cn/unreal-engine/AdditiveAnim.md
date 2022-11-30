# UE中的叠加动画

## 叠加混合
动画Pipeline的核心三要素是采样，混合，后处理。混合最常见的是线性混合，而叠加混合（也有称作加法混合）则是为混合带来一种新的方式。通过引入Difference Clip
形式的动画，即一个Pose到另一个Pose所需的改变，再将其加进普通动画序列，产生一些有趣的姿势和动作变化。

我们令 S= source clip, R = reference clip，那么叠加动画是D = S-R。我们再令要混合的BasePose为T，那么混合出来的结果Outpout = T + D。
个人认为，叠加动画的美妙之处在于,它得到的结果O同时保留了T和S中动画师想要表达的效果，而且T的选取可以和R没有关联。

## UE中设置叠加动画
叠加动画本身的原理很简单，无非就是一组TargetPose减去BasePose得到DeltaPose。在UE里面，通过设置UAnimSequence的AdditiveSettings来开启或者关闭。
而且UAnimSequence开启了叠加设置，那么通过下面方式得到的数据都是经过处理的Delta差值。
```C++
UAnimSequence::GetAnimationPose(OutPose, OutCurve, ExtractionContext)
```
![settings](AdditiveAnim/settings.png ':size=80%')

这是因为在动画编辑器里引擎做了后处理操作，在PostEdit流程中，如果AdditiveSettings发生了变更，会更新动画数据并重新压缩。也就是动画资源本身的叠加与否是离线
操作完成的，并不是运行时计算得到的。当然UE也提供了一个动画节点MakeDynamicAdditive来满足Runtime计算叠加动画的需求。

![PostEdit](AdditiveAnim/PostEdit.png ':size=90%')

## 使用叠加动画
在叠加动画的使用上，相比较一般的绝对动画，没有太大区别，常见的都是在LocalSpace也就是骨骼空间进行计算的。但是如果多个叠加动画混合后再叠加，最终的效果可能会与预期产生偏差，经典的例子就是制作瞄准偏移AimOffset。比如往一个BasePose叠加一个向上瞄准的AO_LookUp让上半身朝向上方。但是如果先叠加一个倾斜动画的BasePose发生倾斜，然后在骨骼空间叠加一个向上的动画，就会得到一个斜向上的结果。如果希望得到倾斜后向上的结果，就需要在MeshSpace（模型空间）中处理Rotation。
![Aimoffset](AdditiveAnim/Aimoffset.png ':size=70%')

> 可以看到在ApplyMeshSpace的时候会先将Pose转换到MeshSpace，完成叠加计算，再转换回来

![Apply](AdditiveAnim/ApplyAdditive.png ':size=80%')
![MeshSpace](AdditiveAnim/MeshSpace.png ':size=80%')
