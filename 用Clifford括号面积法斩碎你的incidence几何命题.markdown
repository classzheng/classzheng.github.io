> 注意：咱事实上并没有把面积法证明器写完，目前完成了自动消元的一小部分，还不会写扩张规则。因为咱的________属性，这个项目的编写格外费时，甚至拖了2个多月，笑笑得了。 它的码风大概是这样

```
#include "bakaford.hpp"
using namespace Bakaford;
int main(){
    Prover P;
    Point A,B,C,S,Ap,Bp,Cp,X,Y,Z;
    P.basepoint(A,"A").basepoint(B,"B").basepoint(C,"C").basepoint(S,"S")
     .collinear(Ap,"A^*",S,A).collinear(Bp,"B^*",S,B).collinear(Cp,"C^*",S,C)
     .intersection(X,"X",A,B,Ap,Bp)
     .intersection(Y,"Y",B,C,Bp,Cp)
     .intersection(Z,"Z",C,A,Cp,Ap)
     .qed();
    std::cout<<Point::bradump(X,Y,Z)<<"&="<<Point::bracket(X,Y,Z)<<".\\\\\n";
    return 0;
}
```

在《[Automated Theorem Proving in Projective Geometry with Bracket Algebra](https://doi.org/10.1360/za2007-37-5-523)》中，Li H.B. 首次定义（存疑）了**三元Clifford括号**为 \[[\cdot]:\mathscr{V\times V\times V\rightarrow F}\]

> 注：这里字体不对，但咱说不清是什么字体。

![img](https://picx.zhimg.com/80/v2-2206863e66598cef174a75b141b70fc8_720w.png?source=ccfced1a)

也就是原文中取n=3的情况

这样的写法也常见于一些外国教材，譬如 [AoPS Volume-1 Chapter-15 Areas](https://artofproblemsolving.com/) 一章中，出现了类似三元Clifford括号的写法，用来表示三角形的面积。

> *EXAMPLE 15-4*  结论 
> $$
> \quad[GCD]=\frac{[ACD]}3=\frac{[ABC]/2}3=\frac{[ABC]}6
> $$

在这里，咱们的三元Clifford括号用来表示三个点向量围成三角形的**有向**面积，计算公式为 

$$
A=(a_x, a_y)\\ B=(b_x, b_y)\\ C=(c_x, c_y)\\  [ABC] = \frac12\left| \begin{array}{cc} a_x & a_y & 1 \\ b_x & b_y & 1 \\ c_x & c_y & 1 \\ \end{array} \right|
$$
同时满足一个简单的性质 \[[ABC]=−[BAC]=[CBA]\]

所以咱们可以通过这条性质，将带符号的有向面积表达式直接转化为无向面积（中学范围内理解的面积，不带符号）表达式，生成可读性极高的证明。

![Figure.⑨](https://picx.zhimg.com/80/v2-18dd83a9a4f28c159c3905140e55f33d_720w.png?source=ccfced1a)

## 0. 扩张规则 (Expansion) 和 收缩规则 (Contraction)

在论文中多次出现的两条规则。扩张规则可以把两个括号的乘积扩张为一个括号多项式，而收缩规则可以把一个括号多项式收缩为一个单项式。这两条规则的使用可以极大地提高证明的可读性和复杂度，同时大幅简化证明过程。  
$$
[\boldsymbol{\mathrm{A_1A_2A_3}}][\boldsymbol{\mathrm{XYZ}}]=\\ [\boldsymbol{\mathrm{XA_2A_3}}][\boldsymbol{\mathrm{YZA_1}}]+[\boldsymbol{\mathrm{XA_3A_1}}][\boldsymbol{\mathrm{YZA_2}}]+[\boldsymbol{\mathrm{XA_1A_2}}][\boldsymbol{\mathrm{YZA_3}}].\quad\text{(Expansion)}
$$
*好在咱不会写收缩规则，所以暂时不写了（*

如果咱们把扩张规则里待求的所有三角形都画出来，会发现刚好可以排成了一个五角星形状的几何图形

![几何太美妙了](https://picx.zhimg.com/80/v2-810d9c3483d8623469a72829b3e2aa7e_720w.png?source=ccfced1a)

这两条规则都是the Grassmann-Plücker relations的推论。这边搬一下Wolfram上相关的词条：

> The Plücker relations are the homogeneous quadratic equations satisfied by the [Grassmann coordinates](https://mathworld.wolfram.com/GrassmannCoordinates.html), also called [Plücker coordinates](https://mathworld.wolfram.com/PlueckerCoordinates.html), of the[Plücker embedding](https://mathworld.wolfram.com/PlueckerEmbedding.html) of a [Grassmannian](https://mathworld.wolfram.com/Grassmannian.html). If  denotes the coordinate indexed by a -element subset  , the relations can be written  They generate the [homogeneous ideal](https://mathworld.wolfram.com/HomogeneousIdeal.html) of the [Grassmannian](https://mathworld.wolfram.com/Grassmannian.html) in its [Plücker embedding](https://mathworld.wolfram.com/PlueckerEmbedding.html).

*这里立个flag，后面再补（* *验证扩张规则并不困难，但感觉咱还没有完全理解。。 。*
$$
\begin{align} \left[XYZ\right]&= -\left[C^*CA\right]\left[AYX\right] + \left[A^*CA\right]\left[CYX\right]\\ &\overset{\text{expand}}=-\left[C^*CA\right]\left[AYX\right]+(\left[CCA\right]\cdot Const+\left[CAA^*\right]\left[YXC\right]+\left[CA^*C\right]\cdot Const)\\ &=-\left[C^*CA\right]\left[AYX\right]+\left[CAA^*\right]\left[YXC\right] \end{align}
$$
显然，  \[[CCA]\text 和[CA^*C]\]都等于0，第二项就不用展开了，直接消去。观察扩张后的多项式，咱们发现括号里的顺序变了。咱手算过好几个例子，结果都差不多，于是有经验事实（其实证明也不难，懒）：

> 推测： 扩张两个含有相同元素的bracket的结果为原bracket.

不过这里扩张的bracket不含有相同元素的话，结果是一个括号多项式。

扩张是论文中很重要的一个规则。后面的Incidence几何消元语句主要依赖于扩张规则。

## 1. Incidence几何基本关系和其对应的消元语句

> HI 关联公理(从属公理) 基本关系为点属于直线.我们也沿用习惯用语，例如点在直线上、直线经过点、 两点的连线、直线相支于一点等等.    ——《吴文俊全集》数学机械化卷IV 

在论文中，Li H.B. 给出了6条针对点的构造规则和6条针对直线的构造规则。咱们暂时只关注incidence几何部分的构点规则，也就是**P1-P3**，有

- **P1**  \[\mathbf X\] 是一个自由点
- **P2**  \[\mathbf X\] 是一个直线上的半自由点（semifree point）
- **P3**  \[\mathbf X\] 是两条（不相同的且非平行的）直线的交点

这三条规则足以构造出大多数incidence几何命题，比如说Desargues Theorem

![img](https://pic1.zhimg.com/80/v2-cc7fb651e59545e9afa6bf974f506fc3_720w.png?source=ccfced1a)

Desargues Theorem

> 两三角形若对应顶点连线共点，则对应边交点共线。

用伪代码把构点规则写出来大概是

```
(P1)Freepoint(A_1)
(P1)Freepoint(B_1)
(P1)Freepoint(C_1)
(P1)Freepoint(P)
(P2)Semifreepoint(A_2, PA_1)
(P2)Semifreepoint(B_2, PB_1)
(P2)Semifreepoint(C_2, PC_1)
(P3)Intersection(A, B_1C_1, B_2C_2)
(P3)Intersection(B, A_1C_1, A_2C_2)
(P3)Intersection(C, B_1A_1, B_2A_2)
```

结论等式\[conc=[ABC]\quad\small(=0)\]  

咱们的目标就是尽可能地对conc进行有意义的分解，最后消去所有非自由点和自由点，此时conc为0.

![img](https://picx.zhimg.com/80/v2-a4202dc8a9de84a7fbdf09c15729a25f_720w.png?source=ccfced1a)

论文中的5. A theorem proving algorithm

对应的消元规则**E1-E3**的理解并不困难，咱们可以试着自行推导一下：

### E1. Eliminate a free point.

考虑自由点 \[\mathbf X\] ，令  \[\mathbf{A,B,C}\] 为出现次数最多的前三个自由点。让  \[\mathbf X\]  延 \[\mathbf{A,B,C}\]  **扩张**，非退化条件是\[[\boldsymbol{\mathrm{ABC}}]\neq0\]. 这条规则并不难理解，扩张只会打乱括号里的顺序，并不影响整个多项式的值。

### E2. Eliminate a semifree point on a line.

考虑自由点 \[\mathbf X\] 在直线  \[\mathbf{AB}\]  上，令  \[\mathbf C\]为不在直线\[\mathbf{AB}\] 出现次数最多的自由点。让  延  **扩张**，非退化条件是\[[\boldsymbol{\mathrm{ABC}}]\neq0\]. 此时注意此时还有隐含的条件\[[\boldsymbol{\mathrm{ABP}}]=0 .\]

### E3. Eliminate the intersection of two lines.

![img](https://pic1.zhimg.com/80/v2-abd04ac240f1e6c35ea8e69af53c8f68_720w.png?source=ccfced1a)

原论文中的E3规则

论文里的E3规则似乎很复杂，但仔细一看分类讨论的情况就大悟了（大雾）

(8) (9) 似乎是对一个复杂的括号多项式运用扩张规则得到的等式，分类的情况是一个括号单项式=0  *这里挖个坑*

因为咱们有简单结论  ，针对 (8) (9) (10) 讨论的情况，咱们可以发现实际上就是讨论  在不同直线上的情况。

![img](https://picx.zhimg.com/80/v2-9a90eda2302e0820fa488d48ee2ba9fd_720w.png?source=ccfced1a)

这些消元规则咱都仔细检查并检验过了，误差控制在1e-16范围内

式 (11) 咱还不太理解，*好在咱可以直接暴力枚举所有情况*😈 后面会提到为什么咱可以仅通过枚举来消元的

```
std::vector<int> psig={1,2,3};
do {
	Point ai1, ai2, aj1, aj2, ak1, ak2;
	if(psig[0]==1) ai1=p1, ai2=p2;
	if(psig[0]==2) ai1=p3, ai2=p4;
	if(psig[0]==3) ai1=is, ai2=si;
	
	if(psig[1]==1) aj1=p1, aj2=p2;
	if(psig[1]==2) aj1=p3, aj2=p4;
	if(psig[1]==3) aj1=is, aj2=si;
	
	if(psig[2]==1) ak1=p1, ak2=p2;
	if(psig[2]==2) ak1=p3, ak2=p4;
	if(psig[2]==3) ak1=is, ak2=si;
	if(std::fabs(Point::bracket(p,is,si)-
				(Point::bracket(ai1,aj1,aj2)*Point::bracket(ai2,ak1,ak2)-Point::bracket(ai2,aj1,aj2)*Point::bracket(ai1,ak1,ak2)))
					<=eps) {
		Polynomial poly;
		poly.set(Bracket(p,is,si))
		  << Monomial{{Bracket(ai1,aj1,aj2),Bracket(ai2,ak1,ak2)},1}
		  << Monomial{{Bracket(ai2,aj1,aj2),Bracket(ai1,ak1,ak2)},-1};
		eliminators.push_back(poly);
	}
} while(std::next_permutation(psig.begin(),psig.end()));
```

依然还是*挖坑待填 (bushi)*

显然，规则 **P3** 可以消去括号内的未知项。通过逐步的 *扩张-消元-收缩* ，最后的结论等式conc会被消去所有未知项，得出  ，则命题得证。

可见，咱们的消元语句也不过是扩张规则的简单运用。然而规则匹配就花了咱整整3个月都没写出来。。。不过，咱们也有平替呢，犯不着去死磕AST啦。

## 3. 洪加威单点例证法和例证思想

洪加威例证法在国内鲜有人提及，并且常常跟张景中的多点例证法混为一谈。举个例子，证明恒等式  ，咱们有：

> 证明： 
> $$
> \text{根据代数基本定理, 即}\\ \text{任何复系数一元n次多项式方程在复数域上有且仅有n个根(计重数).}\\ \text{假设原等式不成立, 则}x^2-1=(x+1)(x-1)\text{是一个一元二次方程}\\ \text{代入} \begin{cases} x=0\\x=1\\x=-1 \end{cases} \text{, 等式均成立.}\\ \therefore \text{与"方程仅有2个根"矛盾}\\ \therefore \text{命题得证.}\\
> $$

这个是张景中的多点例证法，更多的可见

事实上，咱们并不需要代入3组点，1组就够了，也就是单点例证法：

> 证明： 
> $$
> \text{假设原等式不成立, 则}x^2-1=(x+1)(x-1)\text{是一个一元二次方程}\\ \text{不妨记为} ax^2+bx+c=0\\ \text{显然}|a|， |b|，|c|\leq5\\ \text{将}x=10 \text{代入, }\\ |100a|=|10b+c|\leq10|b|+|c|\leq55\\ \text{已知}a=0\\ \therefore|10b|= |c|\leq5 \Rightarrow b=c=0\\ \therefore \text{命题得证.}\\
> $$
> 来源  [归纳法、演绎法和你所不知道的例证法](https://www.sohu.com/a/226754234_136745)

例证思想允许咱们仅通过构造一组特例来概括总体，这也是Bakaford证明器的一个特色。在咱实现的证明器中，为Prover类编写了一组chain methods来生成点的随机坐标。

```
public: Prover& basepoint(Point &p, std::string tag) {
public: Prover& freepoint(Point &p, std::string tag, const Point &p1, const Point &p2, const Point &p3) {
public: Prover& collinear(Point &p, std::string tag, const Point &p1, const Point &p2) {
public: Prover& intersection(Point &p, std::string tag, const Point &p1, const Point &p2, const Point &p3, const Point &p4) {
```

这组方法可以为第一个参数&p生成一个取值为  的随机坐标，并将p的值压入Prover::conlist且为p构造一个回调函数存入callbacks用于生成对于p的消元规则。

> 这里有一个方法Prover::basepoint，直译过来就是基点。这个概念出现在 复系数质点法证明器 中，这里套用了它的名字。一般来说，basepoints只有三个，并且直接为&p分配不重复的随机坐标，而freepoint方法可以生成异于&p1, &p2, &p3的随机坐标。咱承认这里画蛇添足，但这样写下来其实是有一个过程的，最开始设计程序时就没考虑会写成这样子的。考古了一下，最开始使用随机坐标方法时是半个月前，重构过一版。 *省流：其实就是史山（*

事实上，仅通过构造一组特例就证明结论是完全不严谨的，不过这样咱们也有很多好处，比如

![P3消元规则里的分类讨论，直接计算bracket的值就可以啦](https://pic1.zhimg.com/80/v2-90467376b4ea238c1d90f66883ee1d63_720w.png?source=ccfced1a)

![P3 式(11) 直接算bracket的结果+输出就可以啦](https://picx.zhimg.com/80/v2-5c045411aa69028c16e7108f194f8053_720w.png?source=ccfced1a)

![第 I 版写的代码，不知道到底要包几层，其实给个坐标就可以啦](https://picx.zhimg.com/80/v2-6702bafafd159065b7452572b5596192_720w.png?source=ccfced1a)

## 4. 实战开始    

先用Commit 520f967的代码生成消元规则表的latex代码，得到 
$$
\huge{\boldsymbol{\text{Eliminate Rules:}}} \\ \small\boxed{\begin{cases} [ZCA]&=0.\\ [ZC^*A^*]&=0.\\ [ZBX] &= [CC^*A^*]\cdot0 - [AC^*A^*]\cdot[CBX].\\ [ZBY] &= [CC^*A^*]\cdot[ABY] - [AC^*A^*]\cdot0.\\ [ZB^*X] &= -[C^*CA]\cdot[AB^*X] + [A^*CA]\cdot[CB^*X].\\ [ZB^*Y] &= -[C^*CA]\cdot[AB^*Y] + [A^*CA]\cdot[CB^*Y].\\ [ZXB] &= [CC^*A^*]\cdot0 - [AC^*A^*]\cdot[CXB].\\ [ZXB^*] &= -[C^*CA]\cdot[AXB^*] + [A^*CA]\cdot[CXB^*].\\ [ZXY] &= [CC^*A^*]\cdot[AXY] - [AC^*A^*]\cdot[CXY].\\ [ZXY] &= [CXY]\cdot[AC^*A^*] - [AXY]\cdot[CC^*A^*].\\ [ZXY] &= [C^*CA]\cdot[A^*XY] - [A^*CA]\cdot[C^*XY].\\ [ZXY] &= [C^*XY]\cdot[A^*CA] - [A^*XY]\cdot[C^*CA].\\ [ZXY] &= [XCA]\cdot[YC^*A^*] - [YCA]\cdot[XC^*A^*].\\ [ZXY] &= [XC^*A^*]\cdot[YCA] - [YC^*A^*]\cdot[XCA].\\ [ZYB] &= [CC^*A^*]\cdot[AYB] - [AC^*A^*]\cdot0.\\ [ZYB^*] &= -[C^*CA]\cdot[AYB^*] + [A^*CA]\cdot[CYB^*].\\ [ZYX] &= [CC^*A^*]\cdot[AYX] - [AC^*A^*]\cdot[CYX].\\ [ZYX] &= [CYX]\cdot[AC^*A^*] - [AYX]\cdot[CC^*A^*].\\ [ZYX] &= [C^*CA]\cdot[A^*YX] - [A^*CA]\cdot[C^*YX].\\ [ZYX] &= [C^*YX]\cdot[A^*CA] - [A^*YX]\cdot[C^*CA].\\ [ZYX] &= [YCA]\cdot[XC^*A^*] - [XCA]\cdot[YC^*A^*].\\ [ZYX] &= [YC^*A^*]\cdot[XCA] - [XC^*A^*]\cdot[YCA].\\ [YBC]&=0.\\ [YB^*C^*]&=0.\\ [YAX] &= [BB^*C^*]\cdot[CAX] - [CB^*C^*]\cdot0.\\ [YA^*X] &= -[B^*BC]\cdot[CA^*X] + [C^*BC]\cdot[BA^*X].\\ [YXA] &= [BB^*C^*]\cdot[CXA] - [CB^*C^*]\cdot0.\\ [YXA^*] &= -[B^*BC]\cdot[CXA^*] + [C^*BC]\cdot[BXA^*].\\ [XAB]&=0.\\ [XA^*B^*]&=0.\\ [C^*SC]&=0.\\ [C^*BB^*] &= [SC^*A]\cdot[CBB^*] - [CC^*A]\cdot0.\\ [C^*B^*B] &= [SC^*A]\cdot[CB^*B] - [CC^*A]\cdot0.\\ [C^*AA^*] &= [SC^*B]\cdot[CAA^*] - [CC^*B]\cdot0.\\ [C^*A^*A] &= [SC^*B]\cdot[CA^*A] - [CC^*B]\cdot0.\\ [B^*SB]&=0.\\ [A^*SA]&=0.\\ \end{cases}}\\
$$


*这里忘记加\mathbf了555*

然后就有结论等式 \[{[XYZ]}\]
$$
\begin{align} [XYZ]&\overset{Z}=[CC^*A^*][AXY]-[AC^*A^*][CXY]\\ 	 &\overset{(6)}=[CC^*A^*][AXY]-([CC^*A^*][XYA]+[CA^*A][XYC^*]+[CAA^*][XYA])\\ 	 &\overset{Y}=[CC^*A^*][BB^*C^*][CXA]-\left([CA^*A][BB^*C^*][CAX]-([CAA^*]-[CC^*A^*])[BB^*C^*]\right)\\ 	 &= [CC^*A^*][BB^*C^*][CXA]-[CA^*A][BB^*C^*][CAX]+([CAA^*]-[CC^*A^*])[BB^*C^*]\\ 	 &= \boxed{[BB*C*]}\left([CXA][CC^*A^*]-[CAX][CA^*A]+[CAA^*]-[CC^*A^*]\right)\\ 	 &\simeq[CXA]([CC^*A^*]+[CA^*A])+[CAA^*]-[CC^*A^*]\\ 	 &\overset{X}= \end{align}
$$
好吧咱不会证了，有点吃肝，并且咱怀疑其中是不是有某一项展开错了导致  没有对应的消元规则，等咱把自动消元部分代码写完再来挑战。。。

![咱琪露诺才是最强哒！(bushi)](https://pica.zhimg.com/80/v2-a1ebad30d20c090136739b07c436255a_720w.jpg?source=ccfced1a)

图源 東方幻存神签， [【【东方PV】恋之冻结·琪露诺温泉【IOSYS】】](https://www.bilibili.com/video/BV1Ms411S7AV/?share_source=copy_web&vd_source=af12d57579aea5cecd55dd69e06f9504)

## 2026.08.12 补录

修复AoPS链接显示异常  

## 2026.08.17 补录

- 重构代码
- 增加了自动消元部分功能
- 把所有的“我”改成“咱”，*因为这样子比较チルノ?*（bushi） 这里没有改《吴文俊全集》的原话

![简要输出，若调用.qed(conc, true)的话会详细输出所有生成的消元规则，这里只输出了使用过的消元语句](https://pic1.zhimg.com/80/v2-ed1f5c97c17d40f9af3440f3d3f6cfc4_720w.png?source=ccfced1a)

因为最开始我设定的eps=1e-16太小（虽然也不算小(lll￢ω￢) ）所有很多时候(semi)free points的消元规则不能正确生成——这也确实是直接用一个有理数特例例证的确定。反复斟酌后取了eps=1e-7，然后加入判定提示（就是上面的[xxx]...collinear?）可以手动验证是否误判共线——我懒得再写接口去存构图语句了。

后面可能会断断续续地修缮，在这之前我会先把toy style代码重写一遍并加上注释。