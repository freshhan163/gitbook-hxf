## CSS 省略号相关
[参考代码](https://github.com/freshhan163/hxfJS/blob/master/css/ellipsis.html)
### 单行省略号
```css
p {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}
```
### 多行省略号
重点：```-webkit-line-clamp```，记得兼容性处理：```-moz```
```css
p {
    display: -webkit-box;
    display: -moz-box;
    display: box;
    -webkit-box-orient: vertical;
    -moz-box-orient: vertical;
    box-orient: vertical;
    -webkit-line-clamp: 2;
    -moz-line-clamp: 2;
    overflow: hidden;
    text-overflow: ellipsis;
}
```
### 不定行省略号
### 中间行省略号
