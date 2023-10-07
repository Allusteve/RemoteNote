# 翻译|游戏物理基础|Gaffer On Games - Integration Basics

> 原文地址：[Integration Basics](https://gafferongames.com/post/integration_basics/)

## Introduction
你好，我是Glenn Fiedler，欢迎来到游戏物理的世界

如果你曾经想了解物理模拟是如何在计算机游戏里工作的，那么本系列文章将为你解释它。我认为读者已经熟练掌握C++，对物理和数学也有一定的基础。如果你注意并研究示例源代码，就不再需要其他任何预备知识。

物理模拟的工作原理是根据物理定律做出许多小的预测。这些预测事实上非常的简单，可以直接总结为“这个物体在这里，正在非常快速地向那个方向移动，所以在一个很小的时间段后，这个物体应该出现在那里”。我们使用一个称为积分（Intergration）的数学工具来执行这些预测。

本文章的主题是如何正确地地实现这个数学积分。

## Integrating the Equations of Motion
你可能仍记得在高中或大学的物理课上学到过，力等于质量乘以加速度  
> $F = m * a$  
  
我们可以将式子转换为加速度除以质量。这更直观，因为较重的物体更难投掷。
> $a = F/m$ 

加速度是速度随时间的变化率：
> $dv/dt = a = F/m$

同样地，速度也是位置随时间的变化率
> $dx/dt = v$ 

这意味着，如果我们知道一个物体当前的位置和速度，以及施加到它身上的力，我们可以在未来的某个时刻进行积分，以找到物体那时刻的位置和速度。

## Numerical Integration
对于那些没有在大学里正式学过微分方程的人来说，振作起来，因为你所处的位置几乎和那些学过微分方程的人一样好。这是因为我们不会像在一年级数学中那样解析求解微分方程。相反，我们将进行数值积分来找到解决方案。

数值积分就是这么运作的。首先，给定一个初始的位置和速度，然后在时间轴上向前迈出一小步，找到未来该时刻上的位置和速度。然后重复这一步骤，以小的时间步长向前移动，将上一次计算的结果作为下一次的初始条件。

但是我们如何找到每一步的速度和位置的变化呢？

答案就是运动学方程。

让我们把当前时间称为$t$，时间步长称为$dt$或“delta时间”

我们现在可以用一种所有人都能理解的形式展现运动学方程：  
>$acceleration = force/mass$    
>$change in position = velocity * dt$   
>$change in velocity = acceleration * dt$   

这是非常直观的，因为如果你坐在一辆时速60公里每小时的汽车里，一小时后你就会行驶60公里。同样地，一辆每秒加速度为10公里的汽车在10秒后将以每小时100公里的速度行驶。

当然，上述的逻辑仅在加速度和速度为常量时才成立。但即使它们不是，这仍然是一个相当不错的近似估计。

让我们用代码实现这些逻辑。将一个重1公斤的静止物体放置在原点开始，对物体施加10牛顿的恒定力，并以1秒的时间步长向前推进：
```
double t = 0.0;
    float dt = 1.0f;

    float velocity = 0.0f;
    float position = 0.0f;
    float force = 10.0f;
    float mass = 1.0f;

    while ( t <= 10.0 )
    {
        position = position + velocity * dt;
        velocity = velocity + ( force / mass ) * dt;
        t += dt;
    }
```
输出的结果如下：
```
    t=0:    position = 0      velocity = 0
    t=1:    position = 0      velocity = 10
    t=2:    position = 10     velocity = 20
    t=3:    position = 30     velocity = 30
    t=4:    position = 60     velocity = 40
    t=5:    position = 100    velocity = 50
    t=6:    position = 150    velocity = 60
    t=7:    position = 210    velocity = 70
    t=8:    position = 280    velocity = 80
    t=9:    position = 360    velocity = 90
    t=10:   position = 450    velocity = 100
```
正如你所看到的，在每一个步长我们都知道物体的位置和速度。这就是数值积分

## Explicit Euler
我们刚刚所使用的是一种称为显式欧拉的积分。

为了避免您将来的尴尬，我现在必须指出，Euler 的发音为“Oiler”而不是“yew-ler”，因为它是第一个发现该技术的瑞士数学家 Leonhard Euler 的姓氏。

欧拉积分是最基本的数值积分技术。当变化率在时间步长内保持恒定时，它才 100% 准确。

由于上面的例子中加速度是恒定的，因此速度的积分没有误差。 然而，我们也通过对速度来积分获得位置，而速度由于加速度而增加。这意味着位置的积分存在误差。

这个误差到底有多大呢？ 让我们来看看吧！

对于物体如何在恒定加速度下运动有一个闭合解（解析解）。 我们可以用它来将我们的数值积分位置与精确结果进行比较：
```
    s = ut + 0.5at^2
    s = 0.0*t + 0.5at^2
    s = 0.5(10)(10^2)
    s = 0.5(10)(100)
    s = 500 meters
```

在运动十秒后，物体应该已经移动了500米，但是显式欧拉得到的结果是450米。仅仅10秒后，就有50米的误差了。

这听起来非常非常糟糕，但游戏以如此大的时间步长推进物理模拟并不常见。事实上，游戏的物理计算通常使用更接近显示器帧速率的步长。

在$dt=1/100$的情况下计算会产生更好的结果：
```
    t=9.90:     position = 489.552155     velocity = 98.999062
    t=9.91:     position = 490.542145     velocity = 99.099060
    t=9.92:     position = 491.533142     velocity = 99.199059
    t=9.93:     position = 492.525146     velocity = 99.299057
    t=9.94:     position = 493.518127     velocity = 99.399055
    t=9.95:     position = 494.512115     velocity = 99.499054
    t=9.96:     position = 495.507111     velocity = 99.599052
    t=9.97:     position = 496.503113     velocity = 99.699051
    t=9.98:     position = 497.500092     velocity = 99.799049
    t=9.99:     position = 498.498077     velocity = 99.899048
    t=10.00:    position = 499.497070     velocity = 99.999046
```

正如你所看到的，这是一个非常好的结果。精度对于游戏来说足够了

## Why explicit euler is not (always) so great
对于足够小的时间步长，显式欧拉方法对于恒定的加速度可以得到相当不错的结果，但是如果加速度不是常量呢?

非恒定加速度的一个很好的例子是[弹簧阻尼系统](https://ccrma.stanford.edu/CCRMA/Courses/152/vibrating_systems.html)

在这个系统中，质量附着在弹簧上，它的运动受到某种摩擦的阻尼。有一个力与物体的距离成比例，将其拉向原点，还有一个力，与物体的速度成比例，但方向相反，会减慢速度

加速度在整个时间步长内肯定不是恒定的，而是一个连续变化的函数，它是位置和速度的组合，在时间步长内不断变化。

这是一个[阻尼谐振子(Damped harmonic oscillator)](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的例子，它是一个研究得很好的问题，有一个闭合形式的解，我们可以用来检查我们的数值积分结果


让我们从一个欠阻尼系统开始，在这个系统中，质量在原点附近振荡，同时减速

下面是质量弹簧系统的输入参数：
- 质量：1 Kg
- 初始位置：距离原点1000米
- 胡克定律弹簧系数：$k=15$
- 胡克定律阻尼系数：$b=0.1$


下面是精确解的图表：
![extact_solution](interagtion/damped_extact_solution.png ':size=80%')