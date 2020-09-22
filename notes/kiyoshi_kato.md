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

### [<主页](https://www.wangdekui.com/)

# 物体的运动

## 让物体沿水平方向运动
### 匀速直线运动
---

```c
void Start() {
    x = 0;
    v = 3;
}

void Update() {
    x += v;
}
```

```c
void Start() {
    x = 0;
    v = 10;
}

void Update() {
    x += v;

    if (x > VIEW_WIDTH - CHAR_WIDTH) {
        v = -v;
        x = VIEW_WIDTH - CHAR_WIDTH;
    }
    if (x < 0) {
        v = -v;
        x = 0;
    }
}
```

## 通过键盘控制物体的运动
### 勾股定理
---

```c
// 两个方向加起来，速度大于1，不公平
void Start() {
    x = (VIEW_WIDTH - CHAR_WIDTH) * 0.5f;
    y = (VIEW_WIDTH - CHAR_WIDTH) * 0.5f;
}

void Update() {
    if (GetKey(KeyCode.Left)) {
        x -= PLAYER_VEL;
        if (x < 0f)
            x = 0f;
    }
    if (GetKey(KeyCode.Right)) {
        x += PLAYER_VEL;
        if (x > (float)(VIEW_WIDTH - CHAR_WIDTH))
            x = (float)(VIEW_WIDTH - CHAR_WIDTH));
    }
    if (GetKey(KeyCode.Up)) {
        y -= PLAYER_VEL;
        if (y < 0f)
            y = 0f;
    }
    if (GetKey(KeyCode.Down)) {
        y += PLAYER_VEL;
        if (y > (float)(VIEW_HEIGHT - CHAR_HEIGHT))
            y = (float)(VIEW_HEIGHT - CHAR_HEIGHT));
    }
}
```

```c
// 斜方向的速度保持不变，同时按三个方向，速度会变慢
#define ROOT2 1.41421f
void Update() {
    short bLeftKey, bRightKey;
    short bUpKey, bDownKey;

    bLeftKey = GetKey(KeyCode.Left);
    bRightKey = GetKey(KeyCode.Right);
    bUpKey = GetKey(KeyCode.Up);
    bDownKey = GetKey(KeyCode.Down);

    if (bLeftKey) {
        if (bUpKey || bDownKey)
            x -= PLAYER_VEL / ROOT2;
        else
            x -= PLAYER_VEL;

        if (x < 0)
            x = 0;
    }

    if (bRightKey) {
        if (bUpKey || bDownKey)
            x += PLAYER_VEL / ROOT2;
        else
            x += PLAYER_VEL;
        
        if (x >= (float)VIEW_WIDTH - CHAR_WIDTH)
            x = (float)VIEW_WIDTH - CHAR_WIDTH;
    }

    if (bUpKey) {
        if (bLeftKey || bRightKey)
            y -= PLAYER_VEL / ROOT2;
        else
            y -= PLAYER_VEL;
        
        if (y < 0)
            y = 0;
    }

    if (bDownKey) {
        if (bLeftKey || bRightKey)
            y += PLAYER_VEL / ROOT2;
        else
            y += PLAYER_VEL;
        
        if (y >= (float)VIEW_HEIGHT - CHAR_HEIGHT))
            y = (float)VIEW_HEIGHT - CHAR_HEIGHT);
    }
}
```

## 让物体沿任意方向运动
### 三角函数 正弦 余弦 弧度
---

```c
void Start() {
    fAngle = PI / 6.0f;
    vx = PLAYER_VEL * cosf(fAngle);
    vy = PLAYER_VEL * sinf(fAngle);
}

void Update() {
    x += vx;
    y += vy;

    if ((x < -CHAR_WIDTH) || (x > VIEW_WIDTH) || (y < -CHAR_HEIGHT) || (y > VIEW_HEIGHT)) {
        x = (float)(VIEW_WIDTH - CHAR_WIDTH) / 2.0f;
        y = (flaot)(VIEW_HEIGHT - CHAR_HEIGHT)/  2.0f;
        fAngle += 2.0f * PI / 10.0f;
        if (fAngle > (2.0f * PI))
            fAngle -= 2.0f * PI;
        vx = PLAYER_VEL * cosf(fAngle);
        vy = PLAYER_VEL * sinf(fAngle);
    }
}
```

## 在物体运动中加入重力
### 抛物运动 重力加速度 计算误差 积分
---

```c
// 初中物理无法满足游戏需要，在有加速度的情况下，速度 = 距离 / 时间 不再准确
void Start() {
    y = 200.0f;
    vy = -10.0f;
}

void Update() {
    y += vy;
    vy += GR;
}
```
### 位置　速度　加速度

$ x = \int vdt $

$ v = \int adt $

### $ \Delta x = v\Delta t $

$ y = \int v_ydt $

$ v_y = \int Gdt $

得出

$ v_y = Gt + C_1 $

$ C_1 $ 为积分常数，代表t=0时的速度，即初始速度

得出

$ y = \int v_ydt = \int (Gt + C_1)dt = \dfrac 12Gt^2 + C_1t + C_2 $

$ C_2 $ 为积分常数，代表t=0时的y坐标，即初始位置

### t时刻的速度为

$$
v_y = \sum_{i=1}^tG
$$

### t时刻的位置，上面算法的问题所在

用 i - 1，是因为y加了vy之后，需要给vy加上加速度，计算y时所使用的vy的更新被延后了一次

```c
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
void Update() {
    x = vx * t;
    y = 0.5f * GR * t * t + vy * t + 200.0f;
}
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

## 将北京从一端卷动到另一端
### 镜头位置 卷动幅度 比例关系



## [<主页](https://www.wangdekui.com/)