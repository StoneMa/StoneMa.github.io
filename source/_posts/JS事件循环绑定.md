---
title: JS循环绑定click事件
date: 2018-03-22 14:29:19
tags: [JS,前端]
categories: JS
---
### 遇到的问题：
在开发的上一个Annotation的项目的过程中，遇到了需要绑定鼠标事件的问题，如图中所示，模型上有多个热点，可以通过点击热点实现热点说明的展开与关闭。
![Alt text](./1.png)

热点是数据库中动态查询出来的``List<annotation>``，这样的话，我们在前端就需要循环绑定``onclick``事件。我在迭代前端的对象的过程中，一度按照自己的想法去写（未使用闭包之前的写法）：
```javascript
var anno_data;
function loadAnnos() {
    $.ajax({
        type: 'post',
        url: './queryAnnos.do',
        success: function (data) {
            if (data != null){
                anno_data = data;
                createExsitAnnos();
            }
        }
    })
}
var existAnnos = [];

function createExsitAnnos() {
    for (var j = 0; j < anno_data.length; j++){
        var e_div; // 创建anno content的div
        e_div = document.createElement('div');
        document.body.appendChild(e_div);
        existAnnos.push(e_div);
        e_div.appendChild(sp);
        // 这样绑定click方法不行
        $(e_div).on('click', function hideExistAnno() {
            $(e_div).find("*").toggle('fast');
        })

    }
}
```
导致的结果是对于所有的按钮都没有办法绑定独立的``onclick``事件，鼠标点击最终响应的是最后加载的那个热点。其实这就是常见的JS面试题里的那些东西，只不过换了个形式……自己就掉沟里了。
雷同面试题：下面代码中点击按钮后的内容：
```html
<button id="0">0</button>
<button id="1">1</button>
<button id="2">2</button>
<script> 
$(function(){
    for (var i=0; i<=2; i++) {
        $("#" + i).on("click", function() {
            alert(i);
        });
    };
})
</script>
```
_这段代码如果不仔细看的话会误以为三个按钮点击结果分别为0，1，2。但是运行结果却是3，3，3。_
__我们来分析一下代码执行过程：前三遍循环分别给按钮0，1，2绑定了alert(i)的事件，第四遍循环开始时i=3，不符合i<=2的条件，因此终止循环。这里要注意的是，前三遍循环绑定的是alert(i)事件，而不是alert(0)，alert(1)，alert(2)，因为在绑定的过程中on的事件处理函数里的代码并没有运行，因此在触发click事件之前并不知道i等于几，代码此时只认为i是一个全局变量（实际上i的作用域为最外层的function）。上面分析了，当循环结束时i等于3，因此3个按钮点击均为alert(3)。__

---
回到上面的代码：我起初认为``createExsitAnnos()``方法中``$(e_div)``每次都能获取到当前句柄然后都当前句柄进行事件绑定，但实际并不是这样，在执行：
```javascript
        $(e_div).on('click', function hideExistAnno() {
            $(e_div).find("*").toggle('fast');
        })
```
这段代码的时候，跟上面的1，2，3的例子一样，是对每一个当前句柄绑定了``click``事件``hideExistAnno()``,但是，事件中的具体实现却没有在绑定时执行，也就是说``hideExistAnno()``这个方法是在用户点击鼠标的时候才会被调用，而前面的整个for循环绑定的过程中，都未执行，只是给句柄绑定了一个方法，具体方法实现是在点击的时候才进行，当鼠标点触发的时候，``$(e_div).find("*").toggle('fast');``先在实现方法内部寻找变量，发现并没有``e_div``这个句柄，因此到方法外部寻找句柄，此时的句柄是最后一个for循环的句柄，也就是最后的``annotation``因此所有的鼠标点击都会绑定到最后一个句柄。这就是没有维护好函数的作用域造成的结果。因此我们引入了闭包的思想。

---
### 解决办法：

将上面的事件绑定方法改为使用立即调用函数表达式 去编写：
```javascript
　　(function(value){ 
　　　　//代码块 
　　})(i)//这就是立即调用函数表达式 
```
```javascript
        (function(e){
            $(e).on('click', function hideExistAnno() {
                $(e).find("*").toggle('fast');
            });
        })(e_div)
```
这样写就能实现每个``annotation``的事件绑定，后面括号中的``e_div``就会以参数``e``的形式传递到函数体内。
##### 修改后的代码：
```javascript
var anno_data;
function loadAnnos() {
    $.ajax({
        type: 'post',
        url: './queryAnnos.do',
        success: function (data) {
            if (data != null){
                anno_data = data;
                createExsitAnnos();
            }
        }
    })
}
var existAnnos = [];

function createExsitAnnos() {
    for (var j = 0; j < anno_data.length; j++){
        var e_div; // 创建anno content的div
        e_div = document.createElement('div');
        document.body.appendChild(e_div);
        existAnnos.push(e_div);
        e_div.appendChild(sp);
        // 使用立即执行函数绑定
        (function(e){
            $(e).on('click', function hideExistAnno() {
                $(e).find("*").toggle('fast');
            });
        })(e_div)

    }
}
```

---
### 介绍三种解决循环绑定事件的方法：
#####	1. 第一种、编写一个function，在这个function中返回一个函数 ：
_其中get(0)指的是将jQuery对象转为DOM对象。_ 
```javascript
　　$(function(){ 
　　　　for(var i=1;i<=4;i++){ 
　　　　　　$("#btn"+i).get(0).onclick=btnClick(i); 

　　　　} 
　　}); 
　　var btnClick=function(value){ 
　　　　return function(){ 
　　　　　　alert(value); 
　　　　} 
　　} 
```
#### 2. 第二种、使用立即调用函数表达式 :
```javascript
　　$(function(){ 
　　　　for(var i=1;i<=4;i++){ 
　　　　　　$("#btn"+i).get(0).onclick=(function(value){ 
　　　　　　　　return function(){ 
　　　　　　　　alert(value); 
　　　　　　} })(i); 

　　　　} 
　　}); 
```
#### 3. 第三种、使用jQuery的each函数 ：
```javascript
　　$(function(){ 
　　　　$.each([1,2,3,4],function(index,value){ 
　　　　　　　　$("#btn"+value).get(0).onclick=function(){ 
　　　　　　　　alert(value); 
　　　　　　} 
　　　　}); 
　　}); 
```