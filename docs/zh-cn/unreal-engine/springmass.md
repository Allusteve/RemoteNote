# SimplePhysics - 质点弹簧系统

> SimplePhysics计划，将学过的游戏物理动画知识，用简单，易懂的方式复习或者讲解一遍


## 概述
我们会扩展动画蓝图节点，实现一个简化版的AnimDynamics动画节点来驱动角色武器的链条骨骼，来模拟其物理表现。动画节点的物理模型会使用经典的质点弹簧模型，支持重力，并希望能以简单的参数设计，来达成目标效果的配置。

出于学习的目的，会提供多种不同的Simulation实现，包括显式，半隐式，隐式等各类积分方法，以加深自身对物理模拟的了解。

测试使用的是第三人称模板项目和官方商城的免费角色资源： Paragon - FengMao

## 基础原理
质点弹簧模型，一个游戏物理模拟领域内，经久不衰的，简单且快速的基础模型，广泛地运用在布料，软骨，柔体变形等各种模拟对象上。该模型简单来说，就是将物体的质量离散到一个个质点，质点之间再用弹簧连接起来，来表现物体的弹力，阻尼力等物理模型的特性

粒子之间由弹簧减震器进行连接。每一个弹簧连接两个粒子，并且基于粒子的位置和速度来产生作用力。粒子可以受重力影响，弹簧可以设置为不同的类型，比如拉伸弹簧（stretch springs）、剪切弹簧（Shear Springs）和弯曲弹簧（bend springs）等。

一种比较直观的弹簧质点方案是基于牛顿力学的建模。即，首先基于简单的力学定律建立系统的动力学模型，然后通过各种方案求解微分方程，最终得到弹簧质点系统的状态。

根据牛顿力学，对于含有


## 实现细节
在游戏内的质点-弹簧模型使用中，一般操作的原子对象是骨骼，因此我们会将粒子（质点）与将骨骼进行关联，一个骨骼的Transform会受到一或多个粒子的影响，一般会将位置和旋转分开进行处理。

### 注意点
1. 实现物理模型
2. 物理模拟数据映射骨骼，处理好骨骼旋转
3. 注意模型的末端骨骼

### 建模
1. 骨骼与质点对应
2. 质点数量
3. 质点位置   


### 更新流程
1. 物理条件初始化
2. 物理模拟
3. 物理驱动动画

- 物理模拟的空间
  - 世界空间
  - 模型空间

- 常见积分方法
  - Symplectic Euler
  - Verlet Inergation

- 骨骼驱动
  - 位移
  - 旋转

- 模拟帧率
  - Substep

- 计算力与更新位置
  - 逐个计算力且更新位置
  - 先计算出所有的力，再一起更新位置 

## 最终效果