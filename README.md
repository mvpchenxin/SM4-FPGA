# 一、国密标准概述

　　国家密码局制定了一系列密码学算法，包括SM1/2/3/4等，其中SM2是椭圆曲线公钥密码算法，SM3是密码杂凑算法，SM4是分组加密算法。

上述算法的现行标准如下。标准原文可在“国家标准全文公开系统”查询。

* SM2：
GB/T 32918系列标准：
《GB/T 32918.1-2016 SM2椭圆曲线公钥密码算法 第1部分：总则》
《GB/T 32918.2-2016 SM2椭圆曲线公钥密码算法 第2部分：数字签名算法》
《GB/T 32918.3-2016 SM2椭圆曲线公钥密码算法 第3部分：密钥交换协议》
《GB/T 32918.4-2016 SM2椭圆曲线公钥密码算法 第4部分：公钥加密算法》
《GB/T 32918.5-2017 SM2椭圆曲线公钥密码算法 第5部分：参数定义》

* SM3：
《GB/T 32905-2016 SM3密码杂凑算法》	

* SM4:
《GB/T 32907-2016 SM4分组密码算法》

SM1算法并未公开。

# 二、SM4简介

　　SM4算法是分组加密算法，分组长度和密钥长度均为128bit。SM4密钥扩展算法与加密(解密)算法均采用非线性迭代结构，运算轮数为32轮；加密与解密算法均采用相同的结构，只是对输入轮密钥进行反序变换。

# 2.1 密钥

　　密钥长度为128bit，用矩阵$MK=({MK}_0,{MK}_1, {MK}_2,{MK}_3)$表示，其中列向量${MK}_i \space (i=0,1,2,3)$表示一个字（32bit）。

# 2.2 密钥扩展算法

　　密钥扩展算法用于生成轮密钥，轮密钥用于加密算法每轮非线性迭代的输入。轮密钥用矩阵$({rk}_1,{rk}_2, \dots , {rk}_{31})$表示，其中列向量${rk}_i \space (i=0, \dots ,31)$为一个字，表示第$i$轮迭代所使用的轮密钥。
    
　　在介绍密码扩展算法之前，需要引入轮函数和合成置换函数。

## 2.2.1 轮函数

　　设轮函数输入为$(X_0,X_1,X_2,X_3)$，轮密钥为$rk \in Z$，轮函数定义为：

$$ F(X_0,X_1,X_2,X_3,rk) = X_0 \oplus T (X_1 \oplus X_2 \oplus X_3 \oplus rk) $$

## 2.2.2 合成置换函数

　　T函数由一个非线性变换和线性变换构成。其中非线性变换函数通过查找表（S盒）确定。设32bit输入为$A=(a_0,a_1,a_2,a_3)$，输出为$B=(b_0,b_1,b_2,b_3)$，其中分量$a_i,b_i \space (i=0, \dots 3)$是一个字节，非线性变换表示为：

$$ (b_0,b_1,b_2,b_3)=(Sbox(a_0),Sbox(a_1),Sbox(a_2),Sbox(a_3)) $$

　　每个Sbox都是$16 \times 16$的查找表。经过S盒置换后，还需进行如下线性变换。设输入为B，输出为C，定义线性变换$L$：

$$C=L(B)=B \oplus (B<<2) \oplus (B<<10) \oplus (B<<18) \oplus (B<<24)$$

## 2.2.3 密钥扩展算法

　　已知加密密钥$MK$，生成初始轮密钥$(K_3,K_2,K_1,K_0)=({MK}_3 \oplus {FK}_3, {MK}_2 \oplus {FK}_2, {MK}_1 \oplus {FK}_1,{MK}_0 \oplus {FK}_0)$

　　从128位密钥扩展出1024位的轮密钥：

$for$ $i$ = $0$ $to$ $31$

　　${rk}_i = K_{i+4} = K_i \oplus T'(K_{i+1} \oplus K_{i+2} \oplus K_{i+3} \oplus {CK}_i)$

$next$ $i$

　　该迭代过程如计算图所示。

　　![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210127201508565-221284072.png)

　　从计算图中可以看出数据间的依赖关系，这对于组合逻辑实现非常重要。

　　其中$T'$是合成置换函数$T$中的线性变换$L$替换为$L'(B)=B \oplus (B<<13) \oplus (B<<23)$。参数$CK$、$FK$都是常量，它们的取值通过查表来确定。

　　从总体看，轮密钥是以密钥为唯一变量的函数，若密钥已经确定，则轮密钥也是确定的。因此轮密钥可以预先计算。

# 2.3 反序变换

　　设反序变换的输入为$A=(a_0,a_1,a_2,a_3)$，输出为$B=(b_0,b_1,b_2,b_3)$，反序变换定义为：

$$(b_0,b_1,b_2,b_3) = (a_3,a_2,a_1,a_0)$$

# 2.4 加密算法

　　设明文输入为$(X_3,X_2,X_1,X_0)$，密文输出为$(Y_0,Y_1,Y_2,Y_3)$，密码扩展算法输出轮密钥为$rk$，则加密算法由如下步骤完成：

（1）非线性迭代

$for$ $i$ = $0$ $to$ $31$

　　$X_{i+4} = F(X_i,X_{i+1},X_{i+2},X_{i+3},{rk}_i)$

$next$ $i$    

（2）输出

$$ (Y_0,Y_1,Y_2,Y_3) = R(X_{32},X_{33},X_{34},X_{35}) $$

# 2.5 解密算法

　　解密算法与加密算法结构相同，只是对轮密钥反序变换后再输入算法，即使用$rk'=({rk}_31,{rk}_30, \dots ,{rk}_0)$作为轮密钥。

# 三、实现

## 3.1 流水线总体结构

　　SM4加密算法核心是32轮非线性迭代，在硬件上，可以通过组合逻辑来实现每轮迭代，32轮迭代就形成一串组合逻辑。即第$i$个组合逻辑块的输出作为第$i+1$个组合逻辑块的输入，于是得到如下所示的非流水线实现：

![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210127190023230-1568070710.png)

　　这种设计诚然可以正确工作，但时序路径已经很长，输入到输出延迟较大，系统工作频率不可能做得很高。

　　解决办法是采用流水线思想，通过寄存器划分流水段。设每个组合逻辑块的延时为$d_i \space (i=0,1, \dots ,31)$，非流水线实现中，关键路径延时为$D=\sum_{i=0}^{31}d_i$，而流水线实现中关键路径的延时为$D'=\max_{0 \le i \le 31}d_i + T_{cko} \lt D$，可见时序得到了改善。

![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210127191307186-1822919141.png)

　　引入流水线会带来额外的启动周期，因为初始时刻每个流水段都空闲。本设计中流水线共有32段，因此启动周期需要32个时钟周期：在第32个时钟沿到来时，第一个密文/明文分组从输出端口流出，从第33个时钟周期开始，每一个时钟周期都能输出一个分组。

### 3.2 行为级模型的建立

　　通过高级语言建立算法的行为级模型。这里采用C++来建模，只贴出关键代码。完整代码参见Github仓库。

合成置换函数$T$：

```cpp
/* Transformation function */
inline uint32_t T(uint32_t x) {
    uint32_t B = sbox(x);
    return B ^ rol(B, 2) ^ rol(B, 10) ^ rol(B, 18) ^ rol(B, 24);
}
```

合成置换函数$T'$

```cpp
/* Transformation inverse function */
inline uint32_t T_1(uint32_t x) {
    uint32_t B = sbox(x);
    return B ^ rol(B, 13) ^ rol(B, 23);
}
```

密钥扩展算法：

```cpp
void keyexpand(const uint8_t *mk, uint32_t K[36]) {
    K[0] = btw(mk) ^ FK[0];
    K[1] = btw(mk+4) ^ FK[1];
    K[2] = btw(mk+8) ^ FK[2];
    K[3] = btw(mk+12) ^ FK[3];
    
    for(int i=0; i<32; i++) {
        K[i+4] = K[i] ^ T_1(K[i+1] ^ K[i+2] ^ K[i+3] ^ CK[i]);
    }
}
```

加密/解密算法：

```cpp
void decenc(bool enc, const uint8_t *in, uint8_t *out, uint32_t K[36]) {
    unsigned long X[36] = {0};
    
    X[0] = btw(in);
    X[1] = btw(in+4);
    X[2] = btw(in+8);
    X[3] = btw(in+12);

    for(int i = 0; i<32; i++) {
        X[i+4] = X[i] ^ T(X[i+1] ^ X[i+2] ^ X[i+3] ^ K[enc ? i+4 : 35-i]);
    }
    
    wtb(X[35], out);
    wtb(X[34], out+4);
    wtb(X[33], out+8);
    wtb(X[32], out+12);
}
```

　　建立的行为级模型可为仿真验证提供依据，可以通过对比FPGA计算结果与行为级模型的计算结果，来检验FPGA实现是否正确。

### 3.3 RTL模型的建立

#### 3.3.1 S盒替换的实现

　　S盒置换用于合成置换函数$T$。S盒置换函数是32bit字到32bit字的映射，按照字节粒度逐字节替换。

　　通过ROM实现S盒函数。将输入字节$a_0,a_1,a_2,a_3$作为ROM的地址，将ROM的数据输出作为S盒函数的值。输入4个字节的替换是并行的。

　　![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210128201749973-316683206.png)

#### 3.3.2 流水级的实现

　　每个流水级实现轮函数，轮函数的基本框架由异或门构成，其中合成置换函数$T$的非线性变换由ROM实现，流水线的数据暂存通过寄存器实现。

　　![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210129230002021-437888065.png)

　　$L$为合成置换函数$T$的线性变换，具体可参见Verilog源码，这里不再赘述。

### 3.3.3 顶层模块的实现

![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210129232025601-39920051.png)

|   信号名称   |   描述   |
| ---- | ---- |
|   CLK_i   |   时钟输入   |
|   RST_N_i   |   异步复位输入（下降沿有效）   |
|   MK_i   |   128bit密钥输入   |
|   MK_VALID_i   |   密钥输入是否有效   |
|   DAT_i   |   128bit明文（密文）分组输入   |
|   DAT_VALID_i   |   分组输入是否有效   |
|   DAT_o   |   128bit密文（明文）分组输出   |
|   DAT_READY_o   |   分组输出是否有效   |

顶层模块内部RTL原理图

![](https://img2020.cnblogs.com/blog/2284808/202101/2284808-20210129231811199-1056222967.png)

其中keyexp模块实现密钥扩展算法，decenc模块实现加密/解密非线性迭代算法。

### 3.3.4 顶层模块的参数化

　　通过可变参数实现顶层模块的可配置性。顶层模块既可以实现分组加密，又可以实现分组解密。此外，还可配置成使用固定轮密钥，从而省去keyexp模块的资源。

|   参数名   |   描述  |  默认值 |
| ---- | ---- | ---- |
|   MODE   |   0：加密； 非0：解密  |  |
|   ENABLE_FIXED_RK   |   0：不使用固定轮密钥；非0：使用固定轮密钥  | 0 |
|   FIXED_RK   |   若使用固定轮密钥，指定1024bit轮密钥  | 128’h0 |

　　注：当ENABLE_FIXED_RK==0时，采用FIXED_RK指定的轮密钥，信号MK_i和MK_VALID_i将被忽略；否则，采用MK_i指定的密钥，轮密钥将动态生成。

# 四、FPGA验证

　　在Xilinx ZYNQ-7010平台构建如下基于ILA（集成逻辑分析仪）的测试环境，验证SM4模块。

![](https://img2020.cnblogs.com/blog/2284808/202102/2284808-20210201232942957-2081406219.png)

通过ILA抓取加密模块与解密模块输入输出信号波形，验证其功能。

输入明文分组：128'h $01 \space 23 \space 45 \space 67 \space 89 \space AB \space CD \space EF \space FE \space DC \space BA \space 98 \space 76 \space 54 \space 32 \space 10$

输入密钥为：128'h $01 \space 23 \space 45 \space 67 \space 89 \space AB \space CD \space EF \space FE \space DC \space BA \space 98 \space 76 \space 54 \space 32 \space 10$

![](https://img2020.cnblogs.com/blog/2284808/202102/2284808-20210201232830948-267470004.png)

加密模块输出密文：ciphertext = 128'h $68 \space 1e \space df \space 34 \space d2 \space 06 \space 96 \space 5e \space 86 \space b3 \space e9 \space 4f \space 53 \space 6e \space 42 \space 46$

解密模块输出明文为：DAT_o = 128'h $01 \space 23 \space 45 \space 67 \space 89 \space AB \space CD \space EF \space FE \space DC \space BA \space 98 \space 76 \space 54 \space 32 \space 10$

结果正确。

## 资源占用

![](https://img2020.cnblogs.com/blog/2284808/202102/2284808-20210201200242019-1703551880.png)

![](https://img2020.cnblogs.com/blog/2284808/202102/2284808-20210201200623372-1779157708.png)

## 时序报告（$Fclk=100MHz$）

![](https://img2020.cnblogs.com/blog/2284808/202102/2284808-20210201200452837-391724203.png)

计算得出理论最大工作频率$F_{max} = \frac{1}{T-WNS} = 130 MHz$

## 功耗

![](https://img2020.cnblogs.com/blog/2284808/202102/2284808-20210201200431583-981751464.png)