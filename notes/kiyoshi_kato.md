<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# 游戏开发的数学和物理 加藤洁

### [<主页](/index.html)

# 物体的运动

### 让物体沿水平方向运动

匀速直线运动
x += v

### 通过键盘控制物体的运动

勾股定理
```python
if (left)
    if (up or down)
        x -= Delta / Root2
    else
        x -= Delta
```

### 让物体沿任意方向运动

三角函数 正弦 余弦 弧度

```python
angle = Pi / 6
vx = delta * cos(angle)
vy = delta * sin(angle)

x += vx
y += vy
```

### 在物体运动中加入重力

抛物运动 重力加速度 计算误差 积分
在有加速度的情况下，速度 = 距离 / 时间 不再准确

##### 位置　速度　加速度

$ x = \int vdt $

$ v = \int adt $

$ \Delta x = v\Delta t $

$ y = \int v_ydt $

$ v_y = \int Gdt $

得出

$ v_y = Gt + C_1 $

$ C_1 $ 为积分常数，代表t=0时的速度，即初始速度

得出

$ y = \int v_ydt = \int (Gt + C_1)dt = \dfrac 12Gt^2 + C_1t + C_2 $

$ C_2 $ 为积分常数，代表t=0时的y坐标，即初始位置

##### t时刻的速度为

$$
v_y = \sum_{i=1}^tG
$$

##### t时刻的位置

用 i - 1，是因为y加了vy之后，需要给vy加上加速度，计算y时所使用的vy的更新被延后了一次

```c
// 存在问题
y += vy;
vy += GR
```

$$
y = \sum_{i=1}^tv_y = \sum_{i=1}^tG(i-1) = G \cdot \dfrac 12 t(t=1) = \dfrac 12Gt^2 - \dfrac 12Gt
$$

正确结果为 $ y = \frac 12Gt^2 $

错误结果少了 $ -\frac 12Gt $

```c
// 使用积分，只需要具体时间即可算出位置，不会积累误差
x = vx * t;
y = 0.5f * GR * t * t + vy * t + 200.0f;
```

## 物体随机飞溅
### 随机数 均匀随机数 正态分布

### 均匀随机数
```c++
//正常
Balls[i].vx = rand() * VEL_WIDTH / (float)RAND_MAX - VEL-WIDTH / 2.0f;
Balls[i].vy = rand() * VEL_HEIGHT / (float)RAND_MAX - VEL_HEIGHT / 2.0f - BASE_VEL;

//整数，向正上方物体，同样速度物体增多
Balls[i].vx = ( rand() % (VEL_WIDTH + 1) ) - VEL_WIDTH / 2.0f; 
Balls[i].vx = ( rand() % (VEL_HEIGHT + 1) ) - VEL_HEIGHT / 2.0f - BASE_VEL; 

//产生从a到b的随机数
rand() * (b = a) / (float)RAND_MAX + a;
```

### 正态分布

$$
p(x) = \frac 1{\sqrt{2\sigma^2}}exp\left( -\frac{ \left( x - \mu \right)^2 }{ 2 \sigma^2 } \right)
$$

Box-Muller  

$$
Z_1 = \sqrt{-2ln(a)} cos(2{\pi}b)
$$  


$$
Z_2 = \sqrt{-2ln(a)} sin(2{\pi}b)
$$

```c++
// √-2ln(a)
fRand_r = sqrtf( -2.0f * logf( ( float )( rand() + 1 ) / ( RAND_MAX + 1 ) ) );
// 2πb
fRand_t = 2.0f * PI * ( float )rand() / RAND_MAX;
Balls[i].vx = ( fRand_r * cosf( fRand_t )) * VEL_WIDTH;
Balls[i].vy = ( fRand_r * sinf) fRand_t )) * VEL_HEIGHT - BASE_VEL;
```

## 圆周运动
### 角速度 向心力


```c++
x = ROT_R * cosf( fAngle ) + ( VIEW_WIDTH - CHAR_WIDTH ) / 2.0f;
y = ROT_R * sinf( fAngle ) + ( VIEW_HEIGHT - CHAR_HEIGHT ) / 2.0f;
fAngle += 2.0f * PI / 120f; // 120帧完成一周运动
```

角速度 单位时间内角度的变化量  
$$
\omega = \dfrac \theta t
$$

$$
\omega = \dfrac {2\pi} T
$$

周期 经过时间T后，物体正好转一周 2$\pi$ 是周长，是T经过的角度  

$$
\theta = {2\pi} \dfrac t T
$$

#### 处理加速度

当施加的加速度确定时，求速度与位置使用的公式 积分  

$ x = \int vdt $

$ v = \int adt $

圆周运动需要根据确定的位置来算出加速度，需要使用 微分  

$ v = \dfrac {dx} {dt} $

$ a = \dfrac {dv} {dt} $

物体正在做圆周运动

$ x = r \cdot cos (\omega t) $

$$
v_x = \dfrac {dx} {dt} = \dfrac {d(r \cdot cos( \omega t ))}{dt} = - r\omega \cdot sin(\omega t)
$$

$$
a_x = \dfrac{dv_x}{dt} = \dfrac d{dt} ( -r \omega \cdot sin(\omega t) ) = -r {\omega}^2 \cdot cos(\omega t)
$$

得到
$ a_x = - \omega^2 x $


同理

$ y = r \cdot sin ( \omega t) $

$$
v_y = \dfrac {dy} {dt} = \dfrac {d ( r \cdot sin( \omega t))} {dt} = r \omega \cdot cos ( \omega t)
$$

$$
a_y = \dfrac {dv_y}{dt} =  \dfrac d{dt}{r\omega \cdot cos(\omega t)} = -r{\omega^2} \cdot sin(\omega t)
$$

得到  
$ a_y = - \omega^2 y $

向心力
$ a_x = - \omega^2 x $
$ a_y = - \omega^2 y $

```c++
rx = ROT_R;//初始位置
ry = 0.0f;
vx = 0.0f;//初始速度
vy = ROT_R * ANGLE_VEL;
```

$ x = r \cdot cos(\omega t) $
$ y = r \cdot sin(\omega t) $

$ v_x = - r \omega sin ( \omega t) $
$ v_y = r \omega cos ( \omega t) $

将时间t = 0代入，求出初始的位置和速度


## 微分方程式 数值解法
### 微分方程 数值解法 欧拉法

运动方程  
F = ma  
加速度  
$ a = \dfrac Fm $  
已知位置，速度，加速度关系  
$ v = \dfrac {dx}{dt} $  
$ a = \dfrac {dv}{dt} $  
得到  
$ a = \dfrac d{dt} ( \dfrac {dx}{dt} ) = \dfrac {d^2x}{dt^2}$  
可得  
$ \dfrac {d^2x}{dt^2} = \dfrac Fm$  

最简单的情况，重力

$ \dfrac {d^2x}{dt^2} = \dfrac {mg}m = g $  
在两边对t进行积分  
$ \int { \dfrac {d^2x}{dt^2} dt } = \int g\ dt $
$ \dfrac {dx}{dt} = gt + C_0 $
再次积分
$ \int \dfrac {dx}{dt} dt = \int \ gt + \int\ C_0 $
$ x = \dfrac 1cgt^2 + C_0t + C_1 $

弹力  
$ F = -kx $  
$ \dfrac {d^2x}{dt^2} = \dfrac {-kx}m $  
无法简单的对两边进行积分求解，因为两边都有x，求解后将微分变成了积分，没有实际意义  
使用三角函数求解
$ x = A \cdot sin(\omega t) $
对等式进行微分后得  
$ \dfrac {d^2x}{dt^2} = -{\omega}^2 x $  
发现如果  
$ {\omega}^2 = \dfrac km $  
方程解为  
$ x = A \cdot sin ( \pm \sqrt{ \frac km } \cdot t ) $

计算方法  最简单的欧拉法  

已知位置与速度关系  
$ \dfrac {dx} {dt} = v $  
将微分方程近似为差分方程  
$ \dfrac {\Delta x} {\Delta t} = v $  
表示为  
$ \dfrac {x_n - x_{n-1}} {\Delta t} = v $  
即  
$ x_n = x_{n-1} + v \Delta t$  
联合方程组  
$x_n = x_{n-1} + v \Delta t$  
$v_n = v_{n-1} + a \Delta t$  
第一次近似时计算较为粗略，时间小精度会上升  
比如将$\delta t$设为0.1，进行十次循环，就可以得到很高精度的单位1的结果  

# 卷动

## 将背景从一端卷动到另一端
### 镜头位置 卷动幅度 比例关系

```c++
void Init()
{
    fCamera_x = VIEW_WIDTH / 2.0f;//Camera
    fBack_x = VIEW_WIDTH / 2.0f - fCamera_x;//Background
}

void MoveBack()
{
    //left
    if (GetAsyncKeyState(VK_LEFT)) {
        fCamera_x -= CAMERA_VEL;
        if (fCamera_x < VIEW_WIDTH / 2.0f)
            fCamera_x = VEIW_WIDTH / 2.0f;
    }
    //right
    if (GetAsyncKeyState(VK_RIGHT)) {
        fCamera_x += CAMERA_VEL;
        if (fCamera_x > (float)(PICTURE_WIDTH - VIEW_WIDTH / 2.0f))
            fCamera_x = (float)(PICTURE_WIDTH - VIEW_WIDTH / 2.0f);
    }
    fBack_x = VIEW_WIDTH / 2.0f - fCamera_x;
}
```

#### 三重卷动

```c++
fBack_x1 = VIEW_WIDTH / 2.0f - fCamera_x;
fBack_x2 = (float)(PICTURE_WIDTH2 - VIEW_WIDTH) / (PICTURE_WIDTH1 - VIEW_WIDTH) * fBack_x1;
fBack_x3 = (float)(PICTURE_WIDTH3 - VIEW_WIDTH) / (PICTURE_WIDTH1 - VIEW_WIDTH) * fBack_x1;
```

## 让背景卷动与角色的运动产生联动
### 区域坐标 画面坐标

```c++
// fCamera_x 区域中镜头
// fChara_sx 区域中角色
// fChara_x 画面上角色
// fBack_x 画面上背景
fCamera_x = fChara_sx; //镜头暂时回到角色的位置，画面中央
if (fCamera_x < VIEW_WIDTH / 2.0f) //检查左移边界，如果出了，限制回来
    fCamera_x = VIEW_WIDTH / 2.0f;
if (fCamera_x > PICTURE_WIDTH - VIEW_WIDTH / 2.0f) //检查右移边界
    fCamera_x = PICTURE_WIDTH - VIEW_width / 2.0f;
fChara_x = fChara_sx - fCamera_x + VIEW_WIDTH / 2.0f - CHARA_WIDTH / 2.0f; //利用相对位置，角色可以移动到屏幕两侧
fBack_x = VIEW_WIDTH / 2.0f - fCamera_x;
```

```c++
// 角色在画面中央附近时背景不动，角色靠近画面边缘时背景开始卷动
if (fCamera_x < fChara_sx - SCROLL_DIF) //检查镜头是否靠近了左侧
    fCamera_x = fChara_sx - SCROLL_DIF;
if (fCamera_x > fChara_sx + SCROLL_DIF) //检查镜头是否靠近了右侧
    fCamera_x = fChara_sx + SCROLL_DIF;
```

## 卷动由地图块组合的地图
### 地图 地图块 整数的减法 移位运算 逻辑运算

```c++
fMap_x = VIEW_WIDTH / 2.0f - fCamera_x; // 地图的显示坐标，左上角，相机在画面中心
fMap_y = VIEW_HEIGHT / 2.0f - fCamera_y;
nBaseChip_x = (int) - fMap_x / CHIPSIZE; // 需要绘制的第一个地图块编号
nBaseChip_y = (int) - fMap_y / CHIPSIZE;
fBasePos_x = fMap_x + nBaseChip_x * CHIPSIZE; // 需要绘制的第一个地图块坐标
fBasePos_y = fMap_y + nBaseCHip_y * CHIPSIZE;
nChipNum_x = VIEW_WIDTH / CHIPSIZE + 1 + 1; // 需要绘制的横向地图块数量
nChipNum_y = VIEW_HEIGHT / CHIPSIZE + 1 + 1; // 需要绘制的纵向地图块数量
fChipPos_y = fBasePos_y;
for (i = 0; i < nChipNum_y; i++) {
    fChipPos_x = fBasePos_x;
    for ( j = 0; j < nChipNum_x; j++) {
        DrawMapChip( fChipPos_x, fChipPos_y, nMapData[nBaseChip_y + i][nBaseChip_x + j]);
        fChipPos_x += CHIPSIZE;
    }
    fChipPos_y += CHIPSIZE;
}

//-200 = 200 / 2 - 300
//-100 = 100 / 2 - 150
//2 = - -200 / 100
//1 = - -100 / 100
//fBasepos_x = fMap_x + (int)(-fMap_x)
//fBasepos_y = fMap_y + (int)(-fMap_y)
// 看来只是向上取整数地图块
4 = 200 / 100 + 1 + 1
3 = 100 / 100 + 1 + 1

// 从左上角开始绘制到屏幕外一格地图，浪费了一部分
// 代码根据相机位置，计算绘制多少格子，保证相机显示的区域是有地图的

// 如果CHIPSIZE 为64
nbaseChip_x = (int) - fMap_x / CHIPSIZE;
nbaseChip_y = (int) - fMap_y / CHIPSIZE;
// 可以用移位运算，提速
nbaseChip_x = (int) - fMap_x >> 6;
nbaseChip_y = (int) - fMap_y >> 6;

// 如果CHIPSIZE 为64
fBasePos_x = fMap_x + nBaseChip_x * CHIPSIZE;
fBasePos_y = fMap_y + nBaseCHip_y * CHIPSIZE;
// 可以用位运算，提速，乘法运算和逻辑运算在不同cpu单元中，可以并行
fBasePos_x = -( (int) -fMap_x & 0x3f); // 64求余数 <=> -fMap_x % 64
fBasePos_y = -( (int) -fMap_y & 0x3f);
```

## 波纹式的摇摆卷动
### 波纹扭曲 正弦波 波长 振幅 周期

```c++
// 光栅扫描法
DRAWPOINT v2Points[VIEW_HEIGHT]; // 图形线的位置

for (int i = 0; i < VIEW_HEIGHT; i++) {
    v2Points[i].y = (float)i;
    //v2Points[i].x = //(float)i; // 0.0f // (float)i * 0.5f; // 线性
    //v2Points[i].x = 30.0f * sinf( 2.0f * PI * v2Points[i].y / 200.0f); // 正弦
    v2Points[i].x = 30.0f * sinf( 2.0f * PI * ( fTime / 60.0f - v2Points[i].y / 200.0f)); // 时间
}

```

$$
y = A \cdot sin ( \dfrac {2\pi}\lambda  x)
$$

$$
y = A \cdot sin ( 2 \pi ( \dfrac tT - \dfrac x\lambda ) )
$$

A 振幅 30.0f
$ \lambda $ 波长  200.0f
T 周期 60.0f

## 制作有纵深感的卷动
### 透视 比例计算 梯形

图片是一个梯形，长边卷动快，短边卷动慢 

```c++
// fBase_x 卷送的基准位置，图形最下端的图形线 卷动速度最快的部分
// 其他图形线的x坐标，都是以fBase_x为基准位置，通过一定的比例关系计算得到的

fLineWidth = PIC_WIDTH_UP;
fLineBase = ( PIC_WIDTH_DOWN - PIC_WIDTH_UP ) / 2.0f;
for (int i = 0; i < VIEW_HEIGHT; i++) {
    v2Points[i].y = (float)i;
    v2Points[i].x = fBase_x * ( fLineWidth - VIEW_WIDTH ) / ( PIC_WIDTH_DOWN - VIEW_WIDTH ) - fLineBase;
    fLineWidth += (float) ( PIC_WIDTH_DOWN - PIC_WIDTH_UP ) / VIEW_HEIGHT;
    fLineBase -= (float) ( PIC_WIDTH_DOWN - PIC_WIDTH_UP ) / VIEW_HEIGHT / 2.0f;
}

    // 比例计算
    // x_B fBack_x 卷动的基准位置，最下端的图形线 卷动速度最快的位置
    // x_L fLineBase 梯形图片中需要被绘制的部分 开始的x坐标的变量
    // w_L fLineWidth 梯形图片中需要被绘制的部分 梯形宽度
    // w_V VIEW_WIDTH 画面宽度
    // w_D PIC_WIDTH_DOWN 图形下端宽度
    // w_U PIC_WIDTH_UP 图形上端宽度

    //       w_L - w_V
    //  x = ————————————  x_B - x_L
    //       w_D - w_V

    // 镜头在最左边时 x_B == 0
```

## 透视理论
### 视景体 view frustum 、 近似

显示屏与视点距离为h 
视点原点为[0 0 0] 
将空间中的点[x y z], 显示屏，视点 构造一个三角形 
设显示屏上x坐标为xd 
则 xd / h = x / z 
得 xd = h x / z 
同理 yd = h y / z 

此时x坐标与y坐标，在显示屏上变为了原来的 h/z 倍 
z坐标越大，距离视点的深度越大，倍率越小，显示的物体越小 
z == h 时， 倍率正好为1倍，不会有放大或缩小 

视景体的断面是一个梯形，梯形图片卷动会让人产生纵深感 

# 碰撞检测

## 长方形物体间的碰撞检测
### 矩形 德摩根定律

```c++
bool CheckHit( F_RECT *prcRect1, F_RECT *prcRect2 )
{
    bool nResult = false;
    if ( ( prcRect1->fRight > prcRect2->fLeft ) &&
         ( prcRect1->fLeft  < prcRect2->fRight ) )
    {
        if ( ( prcRect1->fBottom > prcRect2->fTop ) &&
             ( prcRect1->fTop    < prcRect2->fBOttom ) )
        {
            nResult = true;
        }
    }
    return nResult;
}
```

## 圆形与圆形 圆形与长方形物体间的碰撞检测
### 距离 勾股定理 平方比较

#### 圆形与圆形
```c++
bool CheckHit( F_CIRCLE *pcrCircle1, F_CIRCLE *pcrCircle2 )
{
    bool nResult = false;
    float dx, dy; // 位置坐标之差
    float ar; // 两圆半径之和
    float fDistSqr;

    dx = pcrCircle1->x - pcrCircle2->x; // delta x
    dy = pcrCircle1->y - pcrCircle2->y; // delta y
    fDistSqr = dx * dx + dy * dy;
    ar = pcrCircle1->r + pcrCircle2->r;

    if ( fDistSqr < ar * ar)
        nResult = true;

    return nResult;
}

struct F_CIRCLE {
    float x,y; // 圆心
    float r; // 半径
}
```

#### 圆形与长方形

考虑长方形各边与圆形相交 
圆形完全包含长方形 
长方形完全包含圆形 

```c++
// 1，将需要检测的长方形，在上下左右4个方向均向外扩张，扩张的长度为圆半径r，如果扩张后得到新的长方形内包含了圆心坐标，
// 则认为两物体具备碰撞的可能，反之则无碰撞的可能。
// 2，在满足 1 的情况下，如果圆心坐标在长方形以外、扩张后的长方形的左上，左下，右上，右下四个角处，且圆内没有包含长方
// 形最近的顶点，则认为两物体没有碰撞。

int CheckHit( F_RECT *prcRect1, F_CIRCLE *pcrCircle2 )		// 碰撞检测
{
	int				nResult = false;
	float			ar;								// 圆的半径

	// 对长方形进行粗略检测
	if ( ( pcrCircle2->x > prcRect1->fLeft   - pcrCircle2->r ) &&
		 ( pcrCircle2->x < prcRect1->fRight  + pcrCircle2->r ) &&
		 ( pcrCircle2->y > prcRect1->fTop    - pcrCircle2->r ) &&
		 ( pcrCircle2->y < prcRect1->fBottom + pcrCircle2->r ) )
	{
		nResult = true;
		ar = pcrCircle2->r;
		// 物体碰到左端检测
		if ( pcrCircle2->x < prcRect1->fLeft ) {
			// 左上角检测
			if ( ( pcrCircle2->y < prcRect1->fTop ) )
			{
				if ( ( DistanceSqr( prcRect1->fLeft,  prcRect1->fTop,
									pcrCircle2->x, pcrCircle2->y ) >= ar * ar ) ) {
					nResult = false;
				}
			}
			else {
				// 左下角检测
				if ( ( pcrCircle2->y > prcRect1->fBottom ) )
				{
					if ( ( DistanceSqr( prcRect1->fLeft,  prcRect1->fBottom,
										pcrCircle2->x, pcrCircle2->y ) >= ar * ar ) ) {
						nResult = false;
					}
				}
			}
		}
		else {
			// 物体碰到右端检测
			if ( pcrCircle2->x > prcRect1->fRight ) {
				// 右上角检测
				if ( ( pcrCircle2->y < prcRect1->fTop ) )
				{
					if ( ( DistanceSqr( prcRect1->fRight,  prcRect1->fTop,
										pcrCircle2->x, pcrCircle2->y ) >= ar * ar ) ) {
						nResult = false;
					}
				}
				else {
					// 右下角检测
					if ( ( pcrCircle2->y > prcRect1->fBottom ) )
					{
						if ( ( DistanceSqr( prcRect1->fRight,  prcRect1->fBottom,
											pcrCircle2->x, pcrCircle2->y ) >= ar * ar ) ) {
							nResult = false;
						}
					}
				}
			}
		}
	}

	return nResult;
}
```

## 细长形物体与圆形物体间的碰撞检测
### 点与线段的距离 内积 微分

难度在于求最短距离，原理很简单 

首先需要计算圆心到线段的最短距离 
再比较 圆半径+棍半径 和 最短距离，如果最短距离小，则碰撞 

关于F_RECT_CIRCLE的表示，是 
x y 是经过的点
vx vy 是向量带长度

px = ax t + bx 
py = ay t + by 
0 <= t <= 1 
表示的一条线段 
最短时的t 
t = [ ax ( x0 - bx ) + ay ( y0 - by ) ] / ( ax^2 + ay^2 )

```c++
bool CheckHit( F_CIRCLE 8pcrCircle, F_RECT_CIRCLE *prcRectCircle )
{
    bool nResult = false;
    float dx, dy; // 位置坐标之差
    float t;
    float mx, my; // 对应最短距离的坐标
    float ar;     // 两物体的半径之和

    float fDistSqr;

    dx = pcrCircle->x - prcRectCircle->x;       // delta x
    dy = pcrCircle->y - prcRectCircle->y;       // delta y
    t = ( prcRectCircle->vx * dx + prcRectCircle->vy * dy ) / 
        ( prcRectCircle->vx * prcRectCircle->vx + prcRectCircle->vy * prcRectCircle->vy );
    if ( t < 0.0f ) t = 0.0f; // t的下限
    if ( t > 1.0f ) t = 1.0f; // t的上限

    mx = prcRectCircle->vx * t + prcRectCircle->x; // 有最短距离的线段上的坐标
    my = prcRectCircle->vy * t + prcRectCircle->y;
    fDistSqr = ( mx - pcrCircle->x ) * ( mx - pcrCircle->x ) + 
               ( my - pcrCircle->y ) * ( my - pcrCircle->y ); // 距离的平方
    ar = pcrCircle->r + prcRectCircle->r;
    if ( fDistSqr < ar * ar ) {
        nResult = true;
    }

    return nResult;
}
```

#### 如何求出t?

p 令线段距离最短的点 px py  
a 向量 ax ay  
b 向量 bx by  

根据 p = at + b ( 0 <= t <= 1)  

px = ax t + bx  
py = ay t + by  
0 <= t <= 1  

根据勾股定理  
l^2 = ( px - x0 )^2 + ( py - y0 )^2  
    = ( ax t + bx - x0 )^2 + (ay t + by - y0)^2  
    = ax^2 t^2 + 2ax( bx - x0 )t + x0^2 + ay^2 t^2 + 2ay( by - y0 )t + y0^2  
    = ( ax^2 + ay^2 )t^2 + 2{ ax ( bx - x0 ) + ay ( by - y0 )}t + x0^2 + y0^2  

已知 l1 >= 0 且 l2 >= 0 时，如果 l1^2 >= l2^2 则 l1 >= l2  
因此，令l^2 最小的t，就是点到线段的最短距离所对应的t，为了求极限，将等式微分  
d(l^2) / dt = 2 ( ax^2 + ay^2 ) t + 2( ax(bx - x0) + ax(by - y0) )  

要求使极限为0的t的值，需要使l^2取极值，在上式中应该为极小值,进行二阶微分  
d^2(l^2) / dt^2 = 2( ax^2 + ay^2 ) > 0  
等式必然为正，因此函数的2阶微分为正时所取的极值为极小值  
综上，通过 d^2(l^2) / dt 为0的t的值，可以计算出线段与点的最短距离  

d(l^2) / dt = 2 ( ax^2 + ay^2 ) t + 2( ax(bx - x0) + ax(by - y0) ) = 0  
2 ( ax^2 + ay^2 ) t = -2( ax( bx - x0 ) + ay( by - y0 ) )  
所以  t = [ ax ( x0 - bx ) + ay ( y0 - by ) ] / ( ax^2 + ay^2 ) 

程序中  
ax = prcRectCircle->vx
ay = prcRectCircle->vy
dx = pcrCircle->x - prcRectCircle->x;  
dy = pcrCircle->y - prcRectCircle->y;  

## 扇形物体的碰撞检测
### 条件划分 向量的运算 向量的内分点 圆的方程式

挥舞细长物体时，造成了扇形，扇形经常用到 

```c++
int CheckHit( F_CIRCLE *pcrCircle, F_FAN *pfaFan ) // 碰撞检测
{
    int nResult = false;

    float dx, dy; // 位置差分
    float fAlpha, fBeta;
    float fDelta;
    float ar;   // 两半径之和
    float fDistSqr;
    float a, b, c;
    float d;
    float t;

    dx = pcrCircle->x - pfaFan->x;
    dy = pcrcircle->y - pfaFan->y;
    fDistSqr = dx * dx + dy * dy;

    if (fDistSqr < pcrCircle->r * pcrCircle->r)
    {
        nResult = true;
    }
    else
    {
        fDelta = pfaFan->vx1 * pfaFan->vy2 - pfaFan->vx2 * pfaFan->vy1;
        fAlpha = (dx * pfaFan->vy2 - dy * pfaFan->vx2) / fDelta;
        fBeta = (-dx * pfaFan->vy1 + dy * pfaFan->vx1) / fDelta;

        if ( (fAlpha >= 0f) && (fBeta >= 0f))
        {
            ar = pfaFan->r + pcrCircle->r;
            if (fDistSqr < ar * ar)
                nResult = true;
        }
        else
        {
            
        }
    }
}
```

## [<主页](/index.html)