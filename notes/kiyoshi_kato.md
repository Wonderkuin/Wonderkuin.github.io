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

```c

```

## [<主页](https://www.wangdekui.com/)