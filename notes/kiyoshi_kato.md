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

$ x = \int vdt $
$ v = \int adt $

## [<主页](https://www.wangdekui.com/)