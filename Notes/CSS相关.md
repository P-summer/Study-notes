```
//  三角形
{
  width: 0;
  height: 0;
  border-top: 30px solid transient;
  border-left: 30px solid transient;
  border-right: 30px solid transient;
  border-bottom: 30px solid red;
}
// 过长省略 ...
{
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis
}
// 多行省略 
{
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 5;
  -webkit-box-orient: vertical;
  word-break: break-all;
}
```
伪类是操作文档中已有的元素，而伪元素是创建了一个文档外的元素，两者最关键的区别就是这点。此外，为了书写CSS时进行区分，一般伪类是单冒号，如:hover，而伪元素是双冒号::before。  
一般大部分伪类强制使用单冒号，大部分伪元素单冒号和双冒号都可以，但是为了区分，建议按照规范书写。[伪类](https://www.runoob.com/css/css-pseudo-classes.html)  

first-child 伪类来选择父元素的第一个子元素。伪元素：CSS伪元素是用来添加一些选择器的特殊效果。  
### tips提示或者输入框小角标 ###
```
//before和after位置颜色不同
<style>
  .border-triangle-bottom {
    width: 100px;
    height: 30px;
    border: 1px solid #1d9cd6;
    position: relative;
    border-radius: 4px;
  }

  .border-triangle-bottom:after,
  .border-triangle-bottom:before {
    content: "";
    position: absolute;
    width: 0;
    height: 0;
    border: 4px solid transparent;
    border-top-color: #1d9cd6;
    left: 50%;
    margin-left: -4px;
    bottom: -8px;
  }

  .border-triangle-bottom:after {
    border-top-color: #fff;
    bottom: -7px;
  }
</style>
<div class="border-triangle-bottom" draggable="true"></div>
```
