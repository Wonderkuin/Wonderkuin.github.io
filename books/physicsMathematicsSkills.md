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

---

# 物体的运动

### 让物体沿水平方向运动
```
匀速直线运动
x += v
```
---
### 通过键盘控制物体的运动
```python
#勾股定理
if (left)
    if (up or down)
        x -= Delta / Root2
    else
        x -= Delta
```
---
### 让物体沿任意方向运动
```python
#三角函数 正弦 余弦 弧度
angle = Pi / 6
vx = delta * cos(angle)
vy = delta * sin(angle)

x += vx
y += vy
```
---
### 在物体运动中加入重力
抛物运动 重力加速度 计算误差 积分
在有加速度的情况下，速度 = 距离 / 时间 不再准确

位置　速度　加速度 关系

$ x = \int vdt $

$ v = \int adt $

$ \Delta x = v\Delta t $

$ y = \int v_ydt $

$ v_y = \int Gdt $

得出 $ v_y = Gt + C_1 $

$ C_1 $ 为积分常数，代表t=0时的速度，即初始速度

得出 $ y = \int v_ydt = \int (Gt + C_1)dt = \dfrac 12Gt^2 + C_1t + C_2 $

$ C_2 $ 为积分常数，代表t=0时的y坐标，即初始位置

t时刻的速度为 $ v_y = \sum_{i=1}^tG $

用 i - 1，是因为y加了vy之后，需要给vy加上加速度，计算y时所使用的vy的更新被延后了一次

```c
// 存在问题
y += vy;
vy += GR
```

$ y = \sum_{i=1}^tv_y = \sum_{i=1}^tG(i-1) = G \cdot \dfrac 12 t(t=1) = \dfrac 12Gt^2 - \dfrac 12Gt $

正确结果为 $ y = \frac 12Gt^2 $

错误结果少了 $ -\frac 12Gt $

```c
// 使用积分，只需要具体时间即可算出位置，不会积累误差
x = vx * t;
y = 0.5f * GR * t * t + vy * t + 200.0f;
```
---

# 物体随机飞溅

### 均匀随机数
```python
#正常
x = rand() * width / max - width / 2
y = rand() * height / max - height / 2 - base
#向正上方物体增多 用整数多，就不自然了
x = (rand() % (width + 1)) - width / 2
y = (rand() % (height + 1)) - height / 2 - base
#产生从a到b的随机数
rand() * (b - a) / max + a
```
---
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
---
## 圆周运动
#### 角速度 向心力


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
#### 微分方程 数值解法 欧拉法

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


视点                       显示屏      景色

                                         x 
                                xd       |
                                |        |
                                |        |
0_____________H_________________|________|
0_____________Z__________________________



设显示屏上x坐标为xd 
则 xd / h = x / z 
得 xd = h x / z 
同理 yd = h y / z 

此时x坐标与y坐标，在显示屏上变为了原来的 h/z 倍 
z坐标越大，距离视点的深度越大，倍率越小，显示的物体越小 
z == h 时， 倍率正好为1倍，不会有放大或缩小

数学问题为：
用平面去切割代表视景体的四角椎体，求可以得到的图形

另外，如果切割四角锥体的平面与屏幕的上端或下端平行，那么这种情况
就与带有纵深感的卷动相同，屏幕对应的显示区域是一个梯形。
```
——————————
——————————————
——————————————————
——————————————————————


      ——————————
    ——————————————
  ——————————————————
——————————————————————


            ——————————
        ——————————————
    ——————————————————
——————————————————————
```
屏幕上方的图形学会变得疏松，下方的图形学会密集。
游戏世界中还有很多类似这样不易察觉的小问题。
3D游戏中使用的用于表现纵深感的视景体，也只是一种近似模型，多少存在误差。
主要是因为将观察者的视点看成了一个点，人的眼睛其实不是一个点。
用户接近远离屏幕，斜视屏幕，图形的正确性无法保证。
投影给用户的图像有偏差，用户的视觉系统会为了消除不真实感进行高频运动，严重时还会引发3D眩晕。

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
             ( prcRect1->fTop    < prcRect2->fBottom ) )
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

#### 如何求出点到线段最短距离

```
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
t = ( ax * dx + ay * dy ) / ( ax * ax + ay * ay );  

ax ay 是线段起点到终点方向的向量
圆心是线段外一点

这样算出的t并不一定满足 0 <= t <= 1

if (t < 0f)
    t = 0f;
if (t > 1f)
    t = 1f;

取线段两个端点
```

```c#
private static Vector3 NearestToLine(Vector3 start, Vector3 end, Vector3 curr)
{
    Vector3 pos;

    Vector3 line = end - start;
    Vector3 curr2point = curr - start;

    float dot = Vector3.Dot(line, curr2point);

    if (dot < 0)
    {
        pos = start;
    }
    else
    {
        float dotLine = Vector3.Dot(line, line);

        if (dot > dotLine)
        {
            pos = end;
        }
        else
        {
            var k = dot / dotLine;
            pos = start + new Vector3(k * line.x, k * line.y, k * line.z);
        }
    }

    return pos;
}
```

#### 为什么不用矩形，而是用胶囊

```
倾斜的长方形与圆形的碰撞检测会有非常多的条件分支，导致程序很长
过多的分支，不利于CPU速度优化
使用倾斜长方形的效果，会有不合理地方，即使数学上正确
特别是另一个图形擦过长方形四个角时
```

## 扇形物体的碰撞检测
### 条件划分 向量的运算 向量的内分点 圆的方程式

```
挥舞细长物体时，造成了扇形，扇形经常用到

1 如果扇形圆形在圆形内部，则表示碰撞
2 圆心坐标 在扇形内部，则表示碰撞
2' 圆形的圆心坐标在组成扇形的两个向量之间
并且圆形的圆心坐标到扇形的圆心坐标的距离，小于 圆形半径+扇形半径
则表示碰撞
本条件覆盖率条件2中 圆的圆心坐标在扇形内的情况，并且还考虑到了圆
在扇形内且扇形的弧与圆有交叠的特殊情况
3 扇形的两个向量所构成的外侧边缘线段中，只要其中任意一条与圆又交点
就认为碰撞

条件 1 2' 3 满足任意一个，就认为圆形与扇形碰撞
```

```c++
// F_CIRCEL 圆形 一个点 半径
// F_FAN 扇形 一个点 两个向量
int CheckHit( F_CIRCLE *pcrCircle, F_FAN *pfaFan )
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

    // 扇形圆心在圆形内部 一定碰撞
    if (fDistSqr < pcrCircle->r * pcrCircle->r)
    {
        nResult = true;
    }
    else
    {
        fDelta = pfaFan->vx1 * pfaFan->vy2 - pfaFan->vx2 * pfaFan->vy1;
        fAlpha = (dx * pfaFan->vy2 - dy * pfaFan->vx2) / fDelta;
        fBeta = (-dx * pfaFan->vy1 + dy * pfaFan->vx1) / fDelta;

        // 圆心在扇形开口方向
        if ( (fAlpha >= 0f) && (fBeta >= 0f))
        {
            ar = pfaFan->r + pcrCircle->r;
            if (fDistSqr < ar * ar)
                nResult = true;
        }
        else
        {
            a = pfaFan->vx1 * pfaFan->vx1 + pfaFan->vy1 * pfaFan->vy1;
            b = -( pfaFan->vx1 * dx + pfaFan->vy1 * dy);
            c = dx * dx + dy * dy - pcrCircle->r * pcrCircle->r;
            d = b * b - a * c;
            if ( d >= 0.0f } {
                t = ( -b - sqrtf( d ) ) / a;
                if ( ( t >= 0.0f ) && ( t <= 1.0f ) ) {
                    nResult = true;
                }
            }

            a = pfaFan->vx2 * pfaFan->vx2 + pfaFan->vy2 * pfaFan->vy2;
            b = -( pfaFan->vx2 * dx + pfaFan->vy2 * dy);
            c = dx * dx + dy * dy - pcrCircle->r * pcrCircle->r;
            d = b * b - a * c;
            if ( d >= 0.0f } {
                t = ( -b - sqrtf( d ) ) / a;
                if ( ( t >= 0.0f ) && ( t <= 1.0f ) ) {
                    nResult = true;
                }
            }
        }
    }

    return nResult;
}
```

```
圆心在扇形两个向量之间

C 圆心坐标
F 扇形圆心坐标
v1 扇形向量1
v2 扇形向量2

C - F = a * v1 + b * v2
    a > 0 且 b > 0
圆形圆心到扇形圆心的向量， 由扇形两个向量构成，
类似于向量的定比分公式，中的内分公式
假设内分点的位置向量为p，则有
p = (1-t)x + ty (0 <= t <= 1)
而用a表示(1-t) b表示t 公式转化为
p = ax + by ( a >= 0 b >= 0 a + b = 1)

当a b 满足上面形式，向量p就是向量a b 的内分点

如果a + b 不等于1， 只是 a + b > 0
则这个向量长度有变化，不影响方向


C - F = a v1 + b v2

C => Xc Yc
F => Xf Yf
v1 => V1x V1y
v2 => V2x V2y

上式表示为

(Xc - Xf)       (V1x)       (V2x)
            = a         + b
(Yc - Yf)       (V1y)       (V2y)

表示成方程组

Xc - Xf = a V1x + b V2x
Yc - Yf = a V2y + b V2y

如果使用矩阵，还可以写成

(Xc - Xf)       (V1x  V2x) (a)
            =
(Yc - Yf)       (V1y  V2y) (b)

面对上面含有矩阵的等式，乘以 [v1x v2x 换行 v1y v2y] 的逆矩阵

          -1
(V1x V2x)      (Xc - Xf)       (a)
                          =   
(V1y V2y)      (Yc - Yf)       (b)


根据矩阵计算公式

     -1
(a b)        1     (d -b)
        =  _____
(c d)      ad-bc   (-c a)

得到


(a)            1            (V2y  -V2x)  (Xc - Xf)
    = ___________________
(b)    V1x V2y - V2x V1y    (-V1y  V1x)  (Yc - Yf)

设 delta = V1x V2y - V2x V1y

则有

a = { V2y ( Xc - Xf) - V2x ( Yc - Yf) } / delta
b = { -V1x ( Xc - Xf) + V1x ( Yc - Yf) } / delta
```

```
圆与线段的交点判断

圆心坐标 C Xc Yc
圆半径 Rc

线段起点
F  Xf Yf
终点 F + v1

圆的方程式
(x - Xc)^2 + (y = Yc)^2 = Rc^2

线段函数
P = v1 t + F   (0 <= t <= 1)
分解为
x = V1x t + Xf
y = V1y t + Yf
0 <= t <= 1

将线段等式代入圆的方程式
( V1x t + Xf - Xc )^2 + ( V1y t + Yf - Yc )^2 = Rc^2

展开合并为 a x^2 + 2bx + c = 0

a = V1x ^2 + V1y ^2
b = - { ( Xc - Xf) V1x + (Yc = Yf) V1y }
c = ( Xc - Xf) ^2 + (Yc - Yf)^2 - Rc ^2

注意公式是 2b
d = b * b - a * c

d 大于0，则有交点

解为
x = (-b +- sqrt( d )) / a

如果 t >= 0 && t <= 1
则线段在圆上有解
```

# 光线的制作

## 让物体向任意方向旋转 含缩放效果
### 旋转 基向量 向量加法 向量减法

```
目标物体位置固定时的旋转

为什么不用旋转矩阵

知道 起点 终点 投射方向的角度是未知的
如果采用旋转矩阵来倾斜物体
首先连接起点终点得到一条直线
然后求得这条直线与x轴的夹角
通过夹角可以得到一个旋转矩阵并控制物体的旋转

计算过程浪费

通过倾斜的直线先求得角度
再通过角度确定旋转矩阵并控制物体旋转的操作

本质上等同于
将不同的坐标系先转换到包含角度的特殊坐标系 例如极坐标系
然后再重新转换回普通坐标系

然而如果只是为了让物体旋转，这些坐标系之间的转换其实是多余的
而且从工程学的角度看，求直线与X轴的夹角用到反三角函数 atan2
很花时间，应该尽量避免使用


不使用旋转矩阵，具体要如何操作呢？
我们用 基变换 这一方法


x方向的基向量 i (1, 0)
y方向的基向量 j (0, 1)
任意坐标 (x, y) 表示为 xi + yj的形式
这一不更改x y ，只是更改i j 也可以实现物体的旋转
不更改i j，只更改x y，就会破坏i j的坐标正交关系
这种变换成为切变 shear conversion

假设这样得到的一对新的基向量为i' j'
然后进行向新坐标系的坐标变换
P = xi + yj
移动到了
P' = xi' + yj'

可以用分量表示
(P'x)         (i'x)         (j'x)
        =   x           +y 
(P'y)         (i'y)         (j'y)

用矩阵表示为
(P'x)       (i'x    j'x) (x)
       =
(P'y)       (i'y    j'y) (y)

也就是说，只要将新的基向量i' j'的分量表示成为矩阵形式
就可以得到从一般坐标系到新坐标系的变换矩阵
最好背下来
普通的旋转矩阵就是通过这种方式得到的矩阵的一个特例
```

```
得到 i' j'

v2Forward.x = (float) CursorPos.x;
v2Forward.y = (float) CursorPos.y;
v2Side.x = -v2Forward.y;
v2Side.y = v2Forward.x;

// v2Forward 是 i'
// v2Side 是 j'

v2Side.x = -v2Forward.y;
v2Side.y = v2Forward.x;
// 这两句是因为 j' 是 i'顺时针旋转 90度得到的
// 此时 i' j' 的位置关系应当为正交
// 上面式子代表的就是正交的旋转变换的结果

( cos pi/2      - sin pi/2)       (0 -1)
                              =
( sin pi/2        cos pi/2)       (1  0)

使用这个矩阵 将 i' 旋转 90度 得到 j'
(j'x)    (0  -1) (i'x)
      =
(j'y)    (1   0) (i'y)

j'x = -i'y
j'y = i'x

注意这里 y轴 下方为正，因此顺时针是转 90度
```

```
对四边形的四个角坐标进行变换

v1Pos1.x = 0.0f;
v1Pos1.y = 0.0f;
v1Pos2.x = (float) CursorPos.x;
v1Pos2.y = (float) CursorPos.y;
v1Pos3.x =      v2Side.x;
v1Pos3.y =      v2Side.y;
v1Pos4.x = CursorPos.x + v2Side.x;
v1Pos4.y = CursorPos.y + v2Side.y;

已经得到了 i'  j'
对四个角坐标进行变换
(0 0)  (1 0)
(0 1)  (1 1)
使用矩阵
(i'x  j'x)
(i'y  j'y)
对四边形的四个角进行变换
左上角 (0 0)
右上角 (i'x  i'y)
左下角 (j'x  j'y)
右下角 (i'x + j'x  i'y + j'y)
```

```
以任意一点为中心旋转

v2Forward.x = (float)CursorPos.x - OriginPos.x;
v2Forward.y = (float)CursorPos.y - OriginPos.y;
v2Side.x = -v2Forward.y;
v2Side.y = v2Forward.x;

v2Pos1.x = (float)OriginPos.x;
v2Pos1.y = (float)OriginPos.y;
v2Pos2.x = (float)CursorPos.x;
v2Pos2.y = (float)CursorPos.y;
v2Pos3.x = OriginPos.x + v2Side.x;
v2Pos3.y = OriginPos.y + v2Side.y;
v2Pos4.x = CursorPos.x + v2Side.x;
v2Pos4.y = CursorPos.y + v2Side.y;

运算量非常小，很好用
```

## 任意两点间光线投射
### 向量长度 单位向量

目标物体在任意位置时如何实现旋转  

```
首先实现 光线宽度可以根据长度联动变化的程序

v2Forward.x = (float)(CursorPos.x - OriginPos.x);
v2Forward.y = (float)(CursorPos.y - OriginPos.y);
v2Side.x = -v2Forward.y / 2.0f;
v2Side.y = v2Forward.x / 2.0f;

v2Pos1.x = OriginPos.x - v2Side.x;
v2Pos1.y = OriginPos.y - v2Side.y;
v2Pos2.x = CursorPos.x - v2Side.x;
v2Pos2.y = CursorPos.y - v2Side.y;
v2Pos3.x = OriginPos.x + v2Side.x;
v2Pos3.y = OriginPos.y + v2Side.y;
v2Pos4.x = CursorPos.x + v2Side.x;
v2Pos4.y = CursorPos.y + v2Side.y;

首先 将光线从起点指向终点的向量放入 v2Forward
也就是说v2Forward与光线的行进方向时平行的
接下来，将v2Forward沿着顺时针方向旋转90度
并将等于其长度一半的向量放入v2Side
也就是说v2Side与光线的行进方向垂直
最后，使用v2Side向量，
将图形的四个角计算出来
由于v2Side代表的是长度为光线长度的一半，并且与光线行进方向垂直的向量
因此无论光线向任何方向投射，都可以通过向量的加减法，得到四个点

核心思路：要使光线可以向任意方向投射，就需要对光线进行旋转
只需要准备一条与光线行进方向垂直的向量，并使用向量间的加减法
```

```
固定宽度的光线

fLength = sqrtf( v2Forward.x * v2Forward.x + v2Forward.y * v2Forward.y );
v2Forward.x /= fLength;
v2Forward.y /= fLength;
v2Side = -v2Forward.y * RAY_WIDTH / 2.0f;
v2Side = v2Forward.x * RAY_WIDTH / 2.0f;

v2Forward向量，是单位向量
```

## 光弯曲处理
### 圆形 圆周长 伪影

```
绘制圆形光线

nDivideNum = (int)( ( 2.0f * PI * r) / 10 );
if (nDivideNum > MAX_DIVIDE_NUM)
    nDivideNum = MAX_DIVIDE_NUM;
fAngleDiff = 2.0f * PI / nDivideNum;
r1 = r - ( RAY_WIDTH / 2.0f );//内圆半径
r2 = r + ( RAY_WIDTH / 2.0f );//外圆半径

fAngle1 = 0.0f;
fAngle2 = fAngleDiff;
for ( i = 0; i < nDivideNum * 4; i += 4 ) {
    v2Pos[i].x = r2 * cosf(fAngle1) + CenterPos.x;
    v2Pos[i].y = r2 * sinf(fAngle1) + CenterPos.y;
    v2Pos[i+1].x = r2 * cosf(fAngle2) + CenterPos.x;
    v2Pos[i+1].y = r2 * sinf(fAngle2) + CenterPos.y;
    v2Pos[i+2].x = r1 * cosf(fAngle1) + CenterPos.x;
    v2Pos[i+2].y = r2 * sinf(fAngle1) + CenterPos.y;
    v2Pos[i+3].x = r1 * cosf(fAngle2) + CenterPos.x;
    v2Pos[i+3].y = r1 * sinf(fAngle2) + CenterPos.y;
    fAngle1 += fAngleDiff;
    fAngle2 += fAngleDiff;
}

问题： 用纹理贴图时 texture mapping时
仅v坐标会被设为有效值，u坐标始终为0
这是由于如果对u坐标也设置一个有效值的话
当圆周变小时，梯形会变得细长，此时由梯形对角线
切分成的三角形会变得极度细长，仅根据三角形三个顶点的uv坐标
无法计算正确的三角形多边形内部的uv坐标，就会发生
伪影 artifacts 现象
```

## 实现带追踪效果的激光
### 左右判定 外积 旋转速度

```
首先，将激光按照某个长度分成很多小节
然后，对每节激光设置不同的方向
从数学角度来看， 最重要的是判断目标物体是在激光行进方向的左侧还是右侧
即：一个点在一个有向线段的左侧还是右侧

// 叉乘 外积
if ( (v2Forward.x * v2Aim.y - v2Forward.y * v2Aim.x) > 0f )
    fAngle += ANGLE_SPEED;
else
    fAngle -= ANGLE_SPEED;

如果不考虑位置关系，会造成不自然的扭动，一会儿左一会儿右


// 以激光为起点，指向追踪目标鼠标指针的向量v2Aim，变成单位向量
fLength = sqrtf( v2Aim.x * v2Aim.x + v2Aim.y * v2Aim.y );
v2Aim.x /= fLength;
v2Aim.y /= fLength;

// 计算出外积
fCross = v2Forward.x * v2Aim.y - v2Forward.y * v2Aim.x;
fAngle += ANGLE_SPEED * fCross / SEGMENT_LEN;

// 利用正弦值，fCross是 sin angle

// v2Forward 向量的长被设置为 SEGMENT_LEN
v2Forward.x = SEGMENT_LEN * cosf( fAngle );
v2Forward.y = SEGMENT_LEN * sinf( fAngle );
```

## 进阶 绘制大幅度弯曲的曲线时的难点
### 曲率 曲线的粗细 插值曲线 反射

```
绘制曲线时，必须要考虑曲线的粗细问题
本节来探究这一问题的难点

上一节中，在曲线的行进方向建立垂直向量(法线向量)
然后将其加到曲线的位置向量，通过这种方法实现了有宽度的曲线的绘制。

这种方法在曲线的弯曲程度(曲率)不是很大时没什么问题，但是曲率增大到一定程度后，多边形会因为
严重扭曲而大量重叠 如果光线是半透明绘制的话，光线的一部分就会变得特别亮


```

---