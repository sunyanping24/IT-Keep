*这本书为什么第一章要说明这个问题，那当然是因为是有必要啊。。。*

**CSS和文档的关系简单一句话来说：有了文档才有了CSS的存在价值**

## css的历史

在早期1990-1993年间，HTML是一个很有限的语言，几乎完全由描述段落、超链接和标题的结构化元素组成。原本HTML只是要作为以重结构化标记语言，用于描述文档的各个部分，这种语言并不关心外观。  

但是由于浏览器的出现，人们认识到万维网的强大，随着人们对网页的认是和要求越来越高，外观已经变的越来越重要。而HTML为了满足人们的需求，提供越来越多新的HTML元素。迫于压力开始出现`<font>` `<big>`这样的标记元素。

以至于后来的HTML一片混乱

## CSS被提出，以一种救星的方式出现

- 丰富的样式
- 易于使用
- 层叠
- 缩减文件大小

之后HTML是一种结构化语言，CSS成为它的补充成为一种样式语言。为了为以后做准备，HTML中的标记元素逐渐被去除。

因此XHTML规范中有很多已经不鼓励使用的元素，这些元素正在逐步从语言中消失。

## 元素的简单介绍（元素是为文档结构的基础）

- 替换和非替换元素
- 元素显示角色（块级元素、行内元素）

### CSS和HTML结合

- 外部样式（link标记）

```
<link rel="stylesheet" type="text/css" href="sheet.css" media="all"></link>
```
1.  样式表并不是HTML文档的一部分，但是仍然会由HTML文档使用。
2. `<link>`必须放在head元素中，但不能放在其他元素内部
3. 样式表中只能有样式规则，不能包含任何其他标记语言
4. `<link>`属性：  
  (1) rel: 代表关系，关系为`stylesheet`  
  (2) type: 标记加载的数据类型  
  (3) href: 样式表的URL，绝对路径、相对路径  
  (4) media: 

- 内部样式

1. `<style></style>`元素可以包含样式表。它在文档中单独出现。  
2. `<style>`元素一定要使用`type`属性，始终是以`<style type="text/css"></style>`开头
3. `<style>`标记中的内容。  
（1）可以使用`@import url('sheet.css')`引入外部样式表  
（2）可以直接写样式,就是具体的样式规则
```
@import url('sheet.css')
body {
  color: red;
}
```

- 内联样式

使用HTML的`style`属性

### CSS样式注释

```
/*xxxx*/

/*xxxx
  xxx
 */
```
