---
title: Markdown语法
date: 2016-08-18 17:41:42
tags: 
- Markdown
categories:
- Tools
- Writting
---

## Markdown简介

> Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档，然后转换成格式丰富的HTML页面。    —— [维基百科](https://zh.wikipedia.org/wiki/Markdown)

## 区块元素
### 段落和换行
一个 Markdown 段落是由一个或多个连续的文本行组成, 它的前后要有一个以上的空行(若某一行只包含空格和tab, 则该行也会被视为空行). 普通段落不该用空格或制表符来缩进. 段落内允许强迫换行(插入换行符). 
如果想强制新起一行则需要先输入2个以上的空格, 然后回车(好像不同编辑器效果不同).
<!-- more -->
### 标题
行首插入 1 到 6 个 `#`, 对应标题 1 到 6. 如: 

    # 这是 H1
    ## 这是 H2
    ###### 这是 H6

也可以在末尾输入对应个`#`, 这纯粹是为了美观.

    # 这是H1 #

### 分割线
在一行中使用三个以上的`*`, '-', `_`来建立分割线, 字符间可以用空格:
    
    * * *
    ***
    ___
    ------------

## 区段元素

### 区块引用
用于引用一段文字.

    > This is a blockquote with two paragraphs. 
    > 
    > Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
    > id sem consectetuer libero luctus adipiscing.
效果: 
> This is a blockquote with two paragraphs. 
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.

也可以偷懒只在整个段落的第一行最前面加上`>`:

    > This is a blockquote with two paragraphs.

    > Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
    id sem consectetuer libero luctus adipiscing.

区块引用还可以嵌套, 只要根据层次加上不同数量的`>`:
    
    > This is the first level of quoting.
    >
    > > This is nested blockquote.
    >
    > Back to the first level.
效果:
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.

区块引用内也可以使用其他的Markdown语法, 如标题, 列表, 代码块等:
    
    > ## 这是一个标题。
    > 
    > 1.   这是第一行列表项。
    > 2.   这是第二行列表项。
    > 
    > 给出一些例子代码：
    > 
    >     return shell_exec("echo $input | $markdown_script");
效果:
> ## 这是一个标题。
> 
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
> 
> 给出一些例子代码：
> 
>     return shell_exec("echo $input | $markdown_script");

### 列表
使用方法为一个列表标记 + 空格 + 内容,
    
无序列表使用`*`, `+`, `-`作为列表标记:

    * Red
    * Green
    * Blue
等同于:
    
    + Red
    + Green
    + Blue
或:
    
    - Red
    - Green
    - Blue
效果(有的编辑器需要在列表前空一行):

- Red
- Green
- Blue

有序列表使用数字加一个英文句点`.`:

    1. Red
    2. Green
    3. Blue
效果:

1. Red
2. Green
3. Blue

需要注意的是, 标记上使用的数字并不会影响最终输入到HTML结果, 因为最终输入的HTML为:

    <ol>
        <li>Red</li>
        <li>Green</li>
        <li>Blue</li>
    </ol>

所以有序列表也可以这么写:

    1. Red
    1. Green
    1. Blue

如果列表见用空行分开, 在输入HTML时列表项目内容会被`<p>`包起来, 所以:

    * Bird
    
    * Magic
效果是:

* Bird

* Magic

如果列表内容有多个段落, 则每个段落的段首必须缩进4个空格或一个tab. 如果列表项内包含引用, 则`>`前也需要缩进. 如果要放代码块的话, 则需要缩进2次. 例:

    *   This is a list item with two paragraphs.

        This is the second paragraph in the list item. You're
        only required to indent the first line. Lorem ipsum dolor
        sit amet, consectetuer adipiscing elit.

        > This is a blockquote
        > inside a list item.
    
            funtion test() {
                console.info("Hello");
            }
可以使用`\`转义列表标识, 防止误渲染.

### 代码
通过缩进4个空格或一个tab就可以建立代码块(前面最好加上一个空行):

	funtion test() {
		console.info("Hello");
	}

	public static void main(String[] args) {
		System.out.print("Hello");
	}
	
有点编辑器支持代码高亮, 则可以使用 三个\` + 语言, 末尾在加三个\`的方式:
    
```java
@Controller
@RequestMapping(value = "/case")
public class CaseController extends AbstractController {

    @RequestMapping(method = RequestMethod.GET)
    public String goPage() {
        return "case";
    }
    
}   
```

## 区块元素
### 链接
Markdown支持两种形式的链接, 行内 和 参考式. 不管哪种, 链接文字都用`[]`包裹.
要建立一个行内式的链接, 只要在方块括号后面紧接着圆括号并插入网址链接即可, 如果你还想要加上链接的 title 文字, 只要在网址后面, 用双引号把 title 文字包起来即可.

    This is [Baidu](http://www.baidu.com/ "This is a title") inline link.
    This is [Baidu](http://www.baidu.com/) inline link without title attribute.

效果:
This is [Baidu](http://www.baidu.com/ "This is a title") inline link. This is [Baidu](http://www.baidu.com/) inline link without title attribute.

如果你要连接到通主机的资源, 则可以使用相对路径:
    
    See my [Home](/) page for details.
    
See my [Home](/) page for details.

参考式的链接是在链接文字的括号后面再接上另一个方括号, 而在第二个方括号里面要填入用以辨识链接的标记:

    This is [an example][id] reference-style link.

接着, 在文件的任意处, 你可以把这个标记的链接内容定义出来:

    [id]: http://example.com/  "Optional Title Here"

效果:
This is [an example][id] reference-style link.
[id]: http://example.com/  "Optional Title Here"

### 强调
使用`*` 或 `_`包裹, :

    *斜体* 或者 _斜体_ 
    **粗体** 或者 __粗体__ 
    ***粗斜体*** 或者 ___粗斜体___ 
效果:
*斜体* 或者 _斜体_ **粗体** 或者 __粗体__ ***粗斜体*** 或者 ___粗斜体___

### 行内代码
使用\`包裹, 如果要在代码区段内插入反引号, 你可以用多个反引号来开启和结束代码区段:

    Use the `print()` function. ``print(`)`` will print a \`.
效果:
Use the `print()` function. ``print(`)`` will print a \`.

### 图片
Markdown 使用一种和链接很相似的语法来标记图片, 同样有行内式 和 参考式:

    ![Markdown Logo](markdown.png)
    ![Markdown Logo](markdown.png "Optional title")
    
    ![Alt text][id]
    
![Alt text](markdown.png "Optional title")
![Alt text][markdownLogo]

[markdownLogo]: markdown.png "Optional title"

Markdown目前无法指定图片的宽高, 如果需要的话, 只能使用`<img>`标签.

    <img src='markdown.png' width='60px'>    
<img src='markdown.png' width='60px'>

### 表格
    | dog | bird | cat |
    | --- | ---- | --- |
    | foo | foo  | foo |
    | bar | bar  | bar |
    | baz | baz  | baz |
    
| dog | bird | cat |
| --- | ---- | --- |
| foo | foo  | foo |
| bar | bar  | bar |
| baz | baz  | baz |

也可以指定列的对齐方向:

    dog | bird | cat
    :---|:----:|---:
    foo | foo  | foo
    bar | bar  | bar
    baz | baz  | baz

dog | bird | cat
:---|:----:|---:
foo | foo  | foo
bar | bar  | bar
baz | baz  | baz

#### 语法说明
- `|`, `-`, `:`之间的多余空格会被忽略
- 默认标题栏居中对齐, 内容居左对齐
- `:-`表示内容和标题栏居左对齐, `:-:`表示内容和标题栏居中对齐, `-:`表示内容和标题栏居右对齐
- 内容和`|`之间的多余空格会被忽略, 每行的第一个和最后一个`|`可以省略, `-`至少要有一个

### 参考
[wowubuntu.com](http://wowubuntu.com/markdown/)
[GLGJing’s Blog](http://glgjing.github.io/blog/2015/04/03/markdown-biao-ge-yu-fa/)


