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

# Markdown

### [<主页](/index.html)
---
___
***
**加粗**
*斜体*
_斜体_
~~删除线.~~
***加粗和斜体***
~~**删除线和加粗**~~
~~*删除线和斜体*~~
~~***加粗, 斜体和删除线***~~
```c
void main()
{
    return 0;
}
```
`code code`

    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code

> Line1  
Line2  
>> Line3  
Line4  

1. Line1
2. Line2
3. Line3

#### 注意结尾的空格
> Block1  
> Block2  
> Block3

* a
- b
+ c

* a
* b
* c
* d
* d
  * f
  * g

1. a
1. b
1. c
1. d

- [x] a
- [ ] b
- [ ] c

| id | content |
| ------ | ------ |
| a | aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa |
| b | bbb |
| c | cc |

  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)

## 内容
## Chapter 1 <a id="chapter-1"></a>
## Chapter 2 <a id="chapter-2"></a>
## Chapter 3 <a id="chapter-3"></a>

这是一个数字脚注[^1].

这是一个带标签的脚注[^label]

[^1]: 这是一个数字脚注
[^label]: 这是一个带标签的脚注

$ e = m c^2 $

## [<主页](/index.html)