**加粗**

1. Line1
2. Line2
3. Line3

> Block1
> Block2  
> Block3

```c
void main()
{
    // 代码段落
    return 0;
}
```

```markdown
### 嵌套markdown语法
```

# h1 标题
## h2 标题
### h3 标题
#### h4 标题
##### h5 标题
###### h6 标题

*渲染为斜体*
_渲染为斜体_

输出的 HTML 看起来像这样:

```html
<em>渲染为斜体</em>
```

```markdown
~~这段文本带有删除线.~~
```

~~删除线.~~

***加粗和斜体***

~~**删除线和加粗**~~

~~*删除线和斜体*~~

~~***加粗, 斜体和删除线***~~


###引用
> Donec massa lacus, ultricies a ullamcorper in, fermentum sed augue.
Nunc augue augue, aliquam non hendrerit ac, commodo vel nisi.
>> Sed adipiscing elit vitae augue consectetur a gravida nunc vehicula. Donec auctor
odio non est accumsan facilisis. Aliquam id turpis in dolor tincidunt mollis ac eu diam.

### 无序列表

```markdown
* 一项内容
- 一项内容
+ 一项内容
```

* a
* b
* c
* d
* d
  * f
  * g

### 有序列表

1. a
2. b
3. c

###自动编号
1. Lorem ipsum dolor sit amet
1. Consectetur adipiscing elit
1. Integer molestie lorem at massa
1. Facilisis in pretium nisl aliquet
1. Nulla volutpat aliquam velit
1. Faucibus porta lacus fringilla vel
1. Aenean sit amet erat nunc
1. Eget porttitor lorem

### 任务列表

- [x] a
- [ ] b
- [ ] c

## 9 代码

### 行内代码

`code code`


### 缩进代码

    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code


## 表格

| id | content |
| ------ | ------ |
| a | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa |
| b | bbb |
| c | cc |


### 链接

<https://www.wangdekui.com>

<wonderkuin@qq.com>

[Assemble](https://assemble.io)

### 定位标记

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


## 脚注

这是一个数字脚注[^1].

这是一个带标签的脚注[^label]

[^1]: 这是一个数字脚注
[^label]: 这是一个带标签的脚注

## 图片
![Lambda](http://www.wangdekui.com/images/lambda.png)
