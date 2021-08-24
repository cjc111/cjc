# 计算几何

​	计算几何是一种更加适合计算机运算的算法。解析几何虽然思路直接，符合我们的直接思维，但是推导出的公式往往不适合计算机的运算。同时，解析几何中建立参数方程来求解的方式，会有大量除法，开根运算，对于敏感的计算机来说误差极大。所以，我们转向优美简洁，只有加减乘除运算的计算几何。但是计算几何也无法完全避免浮点误差，不过相对解析几何来说，可以将其降到最低限度。	

​	计算几何是各种算法中相对独立的一个板块。它思维上的难度不会很大，但是代码量比较大且涉及的知识比较零散。所以，在进入计算几何这个领域前需要强调的是，写题过程要注意代码规范和细节特殊情况。不然面对上百行的代码debug起来就会很头疼。

## 准备

1. 输入输出

   ​	因为在计算几何中会有大量的浮点运算，所以![](http://latex.codecogs.com/svg.latex?double)类型的使用会比较频繁。在用![](http://latex.codecogs.com/svg.latex?scanf)输入时，对于![](http://latex.codecogs.com/svg.latex?double)类型用![](http://latex.codecogs.com/svg.latex?%lf)作为占位符，用![](http://latex.codecogs.com/svg.latex?printf)输出时用![](http://latex.codecogs.com/svg.latex?%lf) 作为占位符。因为![](http://latex.codecogs.com/svg.latex?%lf)在有的系统的![](http://latex.codecogs.com/svg.latex?printf)中是未被定义的，比如大部分来自![](http://latex.codecogs.com/svg.latex?poj)的题目，用了![](http://latex.codecogs.com/svg.latex?%lf)输出就会出问题。

2. 误差控制

   ```c++
   #define EPS 1e-10//根据题目需求修改，
   
   int Judge(double a,double b)//用来判断大小关系，消去误差。
   {
       if(fabs(a-b)<EPS)
           return 0;
       if(a>b)
           return 1;
       else
           return -1;
   }
   ```

## 基本元素

​	实现解题所需的算法之前，我们需要有用来表示几何对象的基本元素。在二维平面中，点和线就作为基本元素。 下面介绍如何用代码表示这些元素。

### 点和向量

​	平面中最简单的元素点![](http://latex.codecogs.com/svg.latex?(x,y)),可以用结构体来实现。另外为方便运算，这里给出点运算中对于运算符号的重载。


```c++
sturct Point
{
	double x，y;//二维平面上的点
    Point operator + (Point p) {return Point(x+p.x,y+p.y);}
    Point operator - (Point p) {return Point(x-p.x,y-p.y);}//实现两个点坐标的相加和相减
   
    Point operator * (double a) {return Point(x*a,y*a);}
    Point operator / (double a) {return Point(x/a,y/a);}//实现点和常量的乘除。
}
```

​	向量在在坐标里，也可以使用一个点来定义。所以可以用和点完全相同的数据结构来表示向量。

```c++
typedef Point Vector;
```

### 线段和线

​	两个点可以构成一个线段，所以我们也是用一个含有两个点的结构体来表示线段。

```c++
struct Segment//线段
{
    Point p1,p2;
    Segment(Point p1=0,Point p2=0):p1(p1),p2(p2){}//构造函数
};
```

​	同理两个点可以确定一条直线，所以直线的表示和线段的表示是一样的。我们用不同的名字加以区分。

```c++
typedef Segment Line;
```



## 点积和叉积

​	点积和叉积是计算几何种最基础，也是最重要的一个工具。虽然点积叉积很简单，但是其应用非常灵活，用得好会有极大简化过程的作用。

###  定义

#### 点积

![](http://latex.codecogs.com/svg.latex?\vec{A}({x_1},{y_1})\cdot\vec{B}({x_2},{y_2})=\left|\vec{A}\right|\left|\vec{B}\right|\sin\alpha={x_1}{y_2}-{x_2}{y_1})

```c++
double Dot(Vector a,Vector b)//向量点积
{
    return a.x*b.x+a.y*b.y;
}
```

#### 叉积

![](http://latex.codecogs.com/svg.latex?\vec{A}({x_1},{y_1})\times\vec{B}({x_2},{y_2})=\left|\vec{A}\right|\left|\vec{B}\right|\cos\alpha={x_1}{x_2}+{y_1}{y_2})

```c++
double Cross(Vector a,Vector b)//向量叉积
{
    return a.x*b.y-a.y*b.x;
}
```

### 应用

#### 判断

​	因为点积叉积里面有三角函数，所以我们可以利用其夹角正负性质坐判断。点积里的![](http://latex.codecogs.com/svg.latex?\cos\alpha)可以用来判断两个向量的方向是否一致，即相对前后位置。叉积里的![](http://latex.codecogs.com/svg.latex?\sin\alpha)可以用来判断两个向量相对上下位置。

![image-20210823155825568](https://github.com/cjc111/cjc/blob/main/image-20210823155825568.png)![image-20210823160013958](https://github.com/cjc111/cjc/blob/main/image-20210823160013958.png)

#### 计算

​	点积叉积里的三角函数值可以在一些计算公式里发挥作用。如叉乘可以计算三角形面积。

## 判断两线段是否相交

​	在判断相交前，我们需要先明确相交的概念。在不同的题目里的相交有着不同的意义，在读题之前就需要区分清楚。

### 规范相交

​	两个线段是否相交可以分成两种情况来讨论。第一种既是下图这种，大多数人第一反应的线段相交。最常见的x型相交，我们可以称之为规范相交。可以给出此处相交的定义：**两条线段有唯一一个不是端点的交点**。

![Untitled Diagram](https://github.com/cjc111/cjc/blob/main/Untitled%20Diagram.png)

​	通过观察，可以发现一个规律两条线段相交的时候，**每条线段的两个端点都在另一条线段的异侧**。通过定义，可以知道这也是规范相交的充要条件。（证明略，可以用反证法或逆推）所以，下面要解决的问题是，如何判断每条线段的两个端点都在另一条线段的异侧。

​	如图（a）有两个线段![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})和![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}}P\mathop{{}}\nolimits_{{22}})。如图（b）若向量![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{22}})和向量![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})分别位于向量![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{21}})的两侧，则可以说明点![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{22}})和点位于![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}})向量的两![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})侧。对于另一个线段也进行同样的判断方法，则可以证明每条线段的两个端点都在另一条线段的异侧。从而可以证明两个线段相交。

![图1](https://github.com/cjc111/cjc/blob/main/image-20210720213235207.png)

​	那么，要如何判断两个向量处于一个向量的异侧，很自然的可以想到高数里面的**向量叉积**。因为叉积的结果里有![](http://latex.codecogs.com/svg.latex?\sin\alpha)。以图（b）为例，如果处于异侧，即以作![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{22}})为![](http://latex.codecogs.com/svg.latex?x)轴，那么上下两个向量于![](http://latex.codecogs.com/svg.latex?x)轴的夹角正弦值乘积，必是负值。

​	所以我们可以得到判断两线段规范相交的公式（以上图为例），即

![](http://latex.codecogs.com/svg.latex?(\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}}}\times\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{21}}})\cdot(\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}}}\times\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{22}}})<0)

和

![](http://latex.codecogs.com/svg.latex?(\overrightarrow{P\mathop{{}}\nolimits_{{21}}P\mathop{{}}\nolimits_{{22}}}\times\overrightarrow{P\mathop{{}}\nolimits_{{21}}P\mathop{{}}\nolimits_{{11}}})\cdot(\overrightarrow{P\mathop{{}}\nolimits_{{21}}P\mathop{{}}\nolimits_{{22}}}\times\overrightarrow{P\mathop{{}}\nolimits_{{21}}P\mathop{{}}\nolimits_{{12}}})<0)

同时成立

```c++
bool Iscross(Point a,Point b,Point c,Point d)//判断线段ab和线段cd是否相交 规范相交
{
    if(Judge(Cross(b-a,c-a)*Cross(b-a,d-a),0.0)==-1&&Judge(Cross(d-c,a-c)*Cross(d-c,b-c),0.0)==-1)
        return 1;
    return 0;
}
```



### 非规范相交

![微信图片_20210815202606](https://github.com/cjc111/cjc/blob/main/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20210815202606.png)	从上图可以看到，非规范相交的情况是特别多的。可以发现，他们都有一个特点，即有一条线段端点在另一条线段上。所以相对于规范相交，这里给出非规范相交的定义，**一条线段至少有一端点在另一条线段上（内部或端点都可以)。**

![image-20210815205210735](https://github.com/cjc111/cjc/blob/main/image-20210815205210735.png)

​	这里以图（a）这种情况为例，并对比规范相交中的图（b），发现两种情况中的四个叉积，只有这一个![](http://latex.codecogs.com/svg.latex?(\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}}}\times\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{21}}}))的正负不一样（为零）。那么，我们可以凭借四个叉积中存在一个为零这一个条件就判断两线段非规范相交吗？答案是否定的。可以看到图（h）和图（g）这两种情况，也存在叉积为零的情况，但是线段并不相交。实际上这里用到的是向量叉积为零的结论，只能说明两向量共线，但并不一定能说明点![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}})在线段![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})内（以上图为例)。

​	那么，如何判断点在线段内。这里我们也可以很自然的联想到高中学过的**点积**。以上图为例，如何判断点![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}})在不在线段![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})上。我们只需要判断![](http://latex.codecogs.com/svg.latex?(\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}}}\cdot\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{21}}}))的符号即可。若等于零，则点![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}})和线段![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})重合；若大于零，则点$P\mathop{{}}\nolimits_{{21}}$在线段![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})外部；若小于零，则点![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}})在线段![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}})内部。以上结论较简单，就不展开说明。

​	所以我们可以得到判断两线段非规范相交的公式，以点![](http://latex.codecogs.com/svg.latex?P\mathop{{}}\nolimits_{{21}})为例，实际上需要判断四次。

![](http://latex.codecogs.com/svg.latex?(\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}}}\times\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{21}}})=0)

和

![](http://latex.codecogs.com/svg.latex?(\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{12}}}\cdot\overrightarrow{P\mathop{{}}\nolimits_{{11}}P\mathop{{}}\nolimits_{{21}}})<=0)

​	同时成立

可以得到以下代码，为了方便理解，下面写成了了四个式子，实际上可以合成一个。

```C++
bool Iscross_n(Point a,Point b,Point c,Point d)//非规范相交
{
    if(Judge(Cross(b-a,c-a)*Cross(b-a,d-a),0.0)==-1&&Judge(Cross(d-c,a-c)*Cross(d-c,b-c),0.0)==-1)
        return 1;
   	else if(Judge(cross(b-a,c-a),0.0)==0&&Judge(dot(c-b,c-a),0.0)<1)
        return 1;
    else if(Judge(cross(b-a,d-a),0.0)==0&&Judge(dot(d-b,d-a),0.0)<1)
        return 1;
    else if(Judge(cross(d-c,a-c),0.0)==0&&Judge(dot(a-c,a-d),0.0)<1)
        return 1;
    else if(Judge(cross(d-c,b-c),0.0)==0&&Judge(dot(b-c,b-d),0.0)<1)
        return 1;
    return 0;
}
```



## 求解多边形面积

### 凸多边形

![image-20210823162254998](https://github.com/cjc111/cjc/blob/main/image-20210823162254998.png)

​	题目给的多边形一般按照逆时针或者顺时针排好序的。凸多边形有n个边，我们就可以把他分成n-2个三角形。所以选定多边形中的一个点作为辅助点，按着顺序，利用叉乘将三角形面积算出来再相加即可。

​	需要注意的是如果点是顺时针方向排序，最后得出的面积是负的，需加绝对值输出。

```c++
double Area(int n,Point p[],int n)//计算多边形面积
{
    double ans=0;
    for(int i=2;i<n;i++)
    {
        ans+=(cross(p[i]-p[1],p[i+1]-p[1]));//这里求出来的是平行四边形面积，最后再除2，减小误差。
    }
    return fabs(ans)/2;
}
```

### 凹多边形

![image-20210823163603254](https://github.com/cjc111/cjc/blob/main/image-20210823163603254.png)

如果是凹多边形，我们是否还可以用上面的方法来求解呢。以B点作为辅助点做三角形。

![image-20210823164645825](https://github.com/cjc111/cjc/blob/main/image-20210823164645825.png)

首先是得到了三角形ACD的面积，这里用黑色表示。

![image-20210823164826578](https://github.com/cjc111/cjc/blob/main/image-20210823164826578.png)

然后三角形BDE因为是负的所以要减去这部分面积，这里用白色表示减去。

![image-20210823164851998](https://github.com/cjc111/cjc/blob/main/image-20210823164851998.png)

​	最后又加上三角形BEA的面积，发现最后求得的面积就是这个多边形的面积。这不是一个巧合。此处涉及到向量的有向面积，证明比较复杂，这里大家记住凹凸多边形的面积求解的代码是一致的。

### 题单

题解还未整理，后续补上

| 序号 | 题号                                                         | 题号                                                         | 知识点                | 难度 |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------- | ---- |
| 001  | [Parallel/Orthogonal](https://vjudge.net/problem/Aizu-CGL_2_A) | Aizu-CGL_2_A                                                 | 内积和外积的使用      | 0    |
| 002  | [Projection](https://vjudge.net/problem/Aizu-CGL_1_A)        | Aizu-CGL_1_A                                                 | 求垂足                | 0    |
| 003  | Intersection                                                 | [Aizu - CGL_2_B ](https://vjudge.net/problem/Aizu-CGL_2_B/origin) | 线关系                | 0    |
| 004  | TOYS                                                         | [POJ - 2318](https://vjudge.net/problem/POJ-2318/origin)     | 点线关系              | ⭐    |
| 005  | Geometric Shapes                                             | [POJ - 3449 ](https://vjudge.net/problem/POJ-3449/origin)    | 线关系                | ⭐⭐⭐  |
| 006  | Rabbit hunt                                                  | [POJ - 2606](https://vjudge.net/problem/POJ-2606/origin)     | 点关系                | ⭐    |
| 007  | 改革春风吹满地                                               | [HDU - 2036](https://vjudge.net/problem/HDU-2036/origin)     | 多边形面积            | ⭐    |
| 008  | Pipe                                                         | [POJ - 1039](https://vjudge.net/problem/POJ-1039/origin)     | 思维+线关系           | ⭐⭐⭐  |
| 009  | Treasure Hunt                                                | [POJ - 1066](https://vjudge.net/problem/POJ-1066/origin)     | 点线关系              | ⭐    |
| 010  | Segments                                                     | [POJ - 3304](https://vjudge.net/problem/POJ-3304/origin)     | 线关系                | ⭐    |
| 011  | Pick-up sticks                                               | [POJ - 2653](https://vjudge.net/problem/POJ-2653/origin)     | 线关系                | ⭐⭐   |
| 012  | An Easy Problem?!                                            | [POJ - 2826](https://vjudge.net/problem/POJ-2826/origin)     | 线关系+毒瘤（我没过） | ⭐⭐⭐⭐ |
| 013  | Line of Sight                                                | [POJ - 2074](https://vjudge.net/problem/POJ-2074/origin)     | 思维+线关系           | ⭐⭐   |
| 014  | The Doors                                                    | [POJ - 1556](https://vjudge.net/problem/POJ-1556/origin)     | 线关系+图论           | ⭐    |
| 015  | Intersecting Lines                                           | [POJ - 1269 ](https://vjudge.net/problem/POJ-1269/origin)    | 线关系                | ⭐⭐   |
| 016  | Lining Up                                                    | [POJ - 1118](https://vjudge.net/problem/POJ-1118/origin)     | 005题进阶             | ⭐⭐   |
| 017  | Dog Distance                                                 | [UVA - 11796](https://vjudge.net/problem/UVA-11796/origin)   | 物理+距离计算         | ⭐⭐   |
| 018  | Segment set                                                  | [HDU - 1558 ](https://vjudge.net/problem/HDU-1558/origin)    | 线关系+并查集         | ⭐⭐   |
| 019  | Pigs can't take a sudden turn                                | [[CSU - 2088](https://vjudge.net/problem/CSU-2088/origin)](http://acm.csu.edu.cn/csuoj/contest/problem?cid=2151&pid=J) | 016题简单版           | ⭐    |
| 020  | Wall                                                         | [POJ - 1113](https://vjudge.net/problem/POJ-1113/origin)     | 凸包                  | ⭐    |
| 021  | Cows                                                         | [POJ - 3348](https://vjudge.net/problem/POJ-3348/origin)     | 凸包                  | ⭐    |
| 022  | Grandpa's Estate                                             | [POJ - 1228 ](https://vjudge.net/problem/POJ-1228/origin)    | 稳定凸包              | ⭐    |
| 023  | The Picnic                                                   | [POJ - 1259 ](https://vjudge.net/problem/POJ-1259/origin)    | 最大空凸包            | ⭐⭐   |
| 024  | Empty Convex Polygons                                        | [HDU - 6219](https://vjudge.net/problem/HDU-6219/origin)     | 最大空凸包            | ⭐⭐   |
| 025  | The Fortified Forest                                         | [POJ - 1873 ](https://vjudge.net/problem/POJ-1873/origin)    | 凸包+优化             | ⭐⭐⭐  |
| 026  | Scrambled Polygon                                            | [POJ - 2007 ](https://vjudge.net/problem/POJ-2007/origin)    | 凸包                  | ⭐    |
| 027  | Two Ants                                                     | [Gym - 102823L](https://vjudge.net/problem/Gym-102823L/origin) | 点边关系+分类         | ⭐⭐⭐⭐ |
| 028  | Convex Hull                                                  | [UVA - 11626 ](https://vjudge.net/problem/UVA-11626/origin)  | 凸包                  | ⭐    |
| 029  | Beauty Contest                                               | [POJ - 2187](https://vjudge.net/problem/POJ-2187/origin)     | 旋转卡壳              | ⭐⭐   |
| 030  | Bridge Across Islands                                        | [POJ - 3608 ](https://vjudge.net/problem/POJ-3608/origin)    | 旋转卡壳              | ⭐⭐   |
| 031  | Rotating Scoreboard                                          | [POJ - 3335](https://vjudge.net/problem/POJ-3335/origin)     | 半平面交              | ⭐⭐   |
| 032  | 线段处理                                                     | [POJ - 1834](https://vjudge.net/problem/POJ-1834/origin)     | 点线关系              | ⭐    |

参考资料 

- 半平面交 https://blog.csdn.net/lonelycatcher/article/details/7973046
- 算法艺术与信息学竞赛 刘汝佳 黄亮
- 挑战程序设计竞赛

​			
