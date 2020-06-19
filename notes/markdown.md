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

# 标题
## 2号
### 3号

# 字体
**加粗**

*斜体*

_斜体_

~~删除线.~~

***加粗和斜体***

~~**删除线和加粗**~~

~~*删除线和斜体*~~

~~***加粗, 斜体和删除线***~~

# 代码
```c
void main()
{
    return 0;
}
```

### 行内代码

`code code`


### 缩进代码

    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code

# 引用
> Line1
Line2
>> Line3
Line4

# 列表
1. Line1
2. Line2
3. Line3

### 注意结尾的空格
> Block1  
> Block2  
> Block3

### 无序列表
* a
- b
+ c

### 无序列表子列表
* a
* b
* c
* d
* d
  * f
  * g

### 自动编号
1. a
1. b
1. c
1. d

### 任务列表

- [x] a
- [ ] b
- [ ] c


# 表格

| id | content |
| ------ | ------ |
| a | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa |
| b | bbb |
| c | cc |


# 链接
```
<https://www.wangdekui.com>

<wonderkuin@qq.com>

[Wangdekui](www.wangdekui.com)
```

# 定位标记

```markdown
## 目录
  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)
```

```markdown
## 内容
## Chapter 1 <a id="chapter-1"></a>
Content for chapter one.

## Chapter 2 <a id="chapter-2"></a>
Content for chapter one.

## Chapter 3 <a id="chapter-3"></a>
Content for chapter one.
```

# 脚注

这是一个数字脚注[^1].

这是一个带标签的脚注[^label]

[^1]: 这是一个数字脚注
[^label]: 这是一个带标签的脚注

# 图片
![Lambda](http://www.wangdekui.com/images/lambda.png)

# Latex公式

$ e = m c^2 $
