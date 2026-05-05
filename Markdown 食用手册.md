## 写在前面
本文大部分来自于[runnoob.com](https://www.runoob.com/markdown/md-tutorial.html) ,仅作学习用
## Markdown标题
Markdown的标题可以通过#+空格+标题内容实现  
examples:  
\# 一级标题  
\## 二级标题  
\### 三级标题  
\#### 四级标题  
\##### 五级标题  
\###### 六级标题  
## Markdown段落
Markdown的换行有2种形式(不过因为obsidian的原因，按照正常输入方法即可)  
一种是直接按下enter进行换行或者是采用空格+回车  
这样的话效果如下  
line1  

line2  
如果是采用空格+空格+回车，则效果如下  
line1  
line2  
## Markdown文字格式
### 字体
\*斜体文本\*      *斜体文本*  
\_斜体文本\_     _斜体文本_  
\*\*粗体文本\*\*     **粗体文本**  
\_\_粗体文本\_\_    __粗体文本__  
\*\*\*粗斜体文本\*\*\*    ***粗斜体文本***  
\_\_\_粗斜体文本\_\_\_    ___粗斜体文本___  
### 分隔线
你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。  你也可以在星号或是减号中间插入空格。  
examples:  
\*\*\*:  
***
### 文本格式修饰元素
##### 删除线
文字的两端加上两个波浪线 ~~ 即可  
examples:  
\~\~我被删除了\~\~  
~~我被删除了  ~~
##### 下划线  
下划线可以通过 HTML 的\<u>标签来实现  
examples:  
\<u>这是带下划线文本\</u>  
<u>这是带下划线文本</u>  
##### 脚注  
脚注是对文本的补充说明。  
格式如下: \[\^要注明的文本]  
examples:  
正文内容[^1]

[^1]: 脚注内容
## Markdown列表
### 无序列表  
使用星号(\*)、加号(+)或是减号(-)作为列表标记，这些标记后面要添加一个空格，然后再填写  
内容  
examples:  
    \* 我前面有点点  
* 我前面有点点  
### 有序列表
有序列表使用数字并加上 . 号+空格+内容来表示。  
examples:  
\1. 第一项  
\2. 第二项  
2. 第一项  
3. 第二项  
### 列表嵌套
列表嵌套只需在子列表中的选项前面添加两个或四个空格即可
examples:
\1. 外项
空格 空格\- 内项
1. 外项
  - 内项

## Markdown区块
Markdown 区块引用是在段落开头使用 > 符号 ，然后后面紧跟一个空格符号  
examples:
\> 区块引用
> 这是区块引用内容  
###### 区块引用兼容嵌套
> 一级引用
> > - 二级引用列表
## Markdown代码
如果是段落上的一个函数或片段的代码可以用反引号把它包起来(\')  
examples:   
`printf()`  
对于代码区块，可以采用三个(\')或者4个空格或者一个制表符  
examples:  
```javascript
$(document).ready(function () {
    alert('RUNOOB');
});
```
	javascript
	$(document).ready(function () {
    alert('RUNOOB');
	});
## Markdown链接
### 普通链接
\[链接名称](链接地址)或者\<链接地址>  
examples
[runnoob.com](https://www.runoob.com/markdown/md-tutorial.html)  
\[runnoob.com\]\(https://www.runoob.com/markdown/md-tutorial.html)
### 高级链接
这个链接用 runoob 作为网址变量 [Runoob][runoob]   

[runoob]: https://www.runoob.com/markdown/md-link.html
```
这个链接用 runoob 作为网址变量 [Runoob][runoob]   
注意，中间至少要换一行
[runoob]: https://www.runoob.com/markdown/md-link.html
```
## Markdown图片
开头一个感叹号 !  
接着一个方括号，里面放上图片的替代文字  
接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的 'title' 属性的文字。  
examples:  
![大小姐](https://w.wallhaven.cc/full/13/wallhaven-136x99.jpg)
同样地,你也可以像网址那样对图片网址使用变量:  
examples:  
网址变量 [test][run]  

[run]:  https://static.jyshare.com/images/runoob-logo.png
```markdown
网址变量 [test][run]  
注意空行
[run]:  https://static.jyshare.com/images/runoob-logo.png
```
html语法同样可行
```
<img src="https://static.jyshare.com/images/runoob-logo.png" width="50%">
```
<img src="https://static.jyshare.com/images/runoob-logo.png" width="50%">  

## Markdown 表格
Markdown 制作表格使用 | 来分隔不同的单元格,使用 - 来分隔表头和其他行。  
注意空行

| 表头  | 表头  |
| --- | --- |
| 单元格 | 单元格 |
| 单元格 | 单元格 |
```
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |
```
相关参数
```
-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。
```
examples:  

| 左对齐 | 右对齐 | 居中对齐 |
| :-- | --: | :--: |
| 单元格 | 单元格 | 单元格  |
| 单元格 | 单元格 | 单元格  |

```
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |
```
