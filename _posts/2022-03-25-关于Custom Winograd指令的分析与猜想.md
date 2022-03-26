---
layout: post
title: "关于Custom Winograd指令的分析与猜想"
date: 2022-03-25 
description: "关于Custom Winograd指令的分析与猜想"
tag: 毕业设计
---   

论文中的winograd算法是利用了一个卷积运算的专用模块进行的，而在这个专用模块有配备了16个专用寄存器，我在猜测如果有可能让自定义的普通指令也能访问到或者说利用到向量寄存器，那么一种可行的汇编描述应该像是下面这样的：

```
# void conv2D_F23(const int8_t* vec1,const int8_t* vect2,int32_t *scalarOut)
# a1=*vect1, a2=*vect2, a3=*scalarOut
# v2=vect1, v4=vect2 , v1=scalarOut
.global conv2D_Winograd_F23:
#配置向量化文件，把128位的向量寄存器分为4个32位的元素
vsetvli t0,4,e32,m1
#初始化输出寄存器为0
vmv.v.i v1,0
#配置向量化文件，把128位的向量寄存器分为16个8位的元素
vsetvli t0,16,e8,m1
#加载卷积input（4*4）
vle.v v2,(a1)
#配置向量化文件，把128位的向量寄存器分出9个8位的元素
vsetvli t0,9,e8,m1
#加载卷积kernel（3*3）
vle.v v4,(a2)
#利用卷积运算优化的Winograd算法，把2*2的32位输出按序写入到一个128位的向量寄存器
conv23 v1,v2,v4
#配置向量化文件，把128位的向量寄存器分为4个32位的元素
vsetvli t0,4,e32,m1
#写入内存
vse.v v1,(a3)
ret

```

论文所提到的Winograd算法主要专注于4*4的输入矩阵，3*3的kernel size，同时卷积运算的stride为1，输出的矩阵大小为2*2，论文中有具体的算法实现描述。目前的问题是我们自定义的指令是否能使用到向量寄存器，退一步来说，就算先用不上Winograd算法，如果能利用向量寄存器来实现卷积运算也可以大幅减少load/store的次数。
