# html基础

## 1.熟悉HTML基本语法

1.1文本标记语言，即HTML（Hypertext Markup Language），是用于描述网页文档的一种标记语言。

1.2一个网页对应于一个HTML文件，HTML文件以.htm或.html为扩展名。可以使用任何能够生成TXT类型源文件的文本编辑来产生HTML文件。 

1.3超文本标记语言标准的HTML文件都具有一个基本的整体结构，即HTML文件的开头与结尾标志和HTML的头部与实体2大部分。有3个双标记符用于页面整体结构的确认。

1.4不区分大小写

## 2.了解常用的html标签

![image-20191221145224915](C:\Users\zhangxiaoliang\Desktop\总结\HTML初级\html常用标签分类.png)

## 3.了解html常用声明

3.1定义和用法

1.<!DOCTYPE> 声明必须是 HTML 文档的第一行，位于 <html> 标签之前。
2.<!DOCTYPE> 声明不是 HTML 标签；它是指示 web 浏览器关于页面使用哪个 HTML 版本进行编写的指令。
3.在 HTML 4.01 中，<!DOCTYPE> 声明引用 DTD，因为 HTML 4.01 基于 SGML。DTD 规定了标记语言的规则，这样浏览器才能正确地呈现内容。
4.HTML5 不基于 SGML，所以不需要引用 DTD。
5.提示：请始终向 HTML 文档添加 <!DOCTYPE> 声明，这样浏览器才能获知文档类型。

![image-20191221145547318](C:\Users\zhangxiaoliang\Desktop\总结\HTML初级\常用的DOCTYPE声明1.png)

![image-20191221145659931](C:\Users\zhangxiaoliang\Desktop\总结\HTML初级\常用的DOCTYPE声明2.png)

## 4.熟练使用常用的属性

| 属性  | 值                 | 描述                                     |
| ----- | ------------------ | ---------------------------------------- |
| class | *classname*        | 规定元素的类名（classname）              |
| id    | *id*               | 规定元素的唯一 id                        |
| style | *style_definition* | 规定元素的行内样式（inline style）       |
| title | *text*             | 规定元素的额外信息（可在工具提示中显示） |

## 5.了解各种标签到底属于什么元素，内联/内联-块级/块级元素

| **内联元素**                                                 |
| ------------------------------------------------------------ |
| 内联元素又名行内元素(inline element)，和其对应的是块元素(block element)，都是html规范中的概念。内联元素的显示，为了帮助理解，可以形象的称为“文本模式”，即一个挨着一个，都在同一行按从左至右的顺序显示，不单独占一行。 |
| **块状元素**                                                 |
| 块元素又名块级元素(block element)，和其对应的是内联元素(inline element)，都是html规范中的概念。块元素和内联元素的基本差异是块元素一般都从新行开始，相邻的块级元素将会在不同行显示。 |
| **如何区分**                                                 |
| 判断的标准有两个：1、是否独占一行；2、是否可以单独为元素设置高度和宽度。 |

| **内联元素**                                                 |
| ------------------------------------------------------------ |
| <span>、<textarea>、 <select>、<input>、<img>、<label>、<a>、<br>、<i>、<b>、<small>、<em>、<strong>、<code>、<cite>、<sup>、<sub>、 <bdo> 、<abbr>、<mark>、<q> |
| **块状元素**                                                 |
| <!DOCTYPE> 、<html>、<head>、<body>、<title>、<meta>、<base>、<style>、<link>、<script>、<noscript><aside>、<footer>、<header>、<nav>、<section>、<div>、<iframe> <table>、<thead>、<tbody>、<tfoot>、<tr>、<td>、<th>、<col>、<colgroup>、<caption> <from>、<button>、<optgroup>、<option>、<fieldset>、<legend> <datalist>、<ul>、<ol>、<li>、<dl>、<dt>、<dd> <dialog>、<meter>、<time>、<keygen>、<output>、<progress>、<!DOCTYPE>、<!-- —>、<hr> <audio>、<video>、<canvas>、<embed>、<figure>、<figcaption>、<source>、<map>、<area>、<object>、<param> <article>、<details>、<summary>、<hgroup>、<h1> - <h6> 、<p>、<br>、<pre>、<blockquote>、<ins>、<del>、<address> |