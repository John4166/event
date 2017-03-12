# 事件模型和事件代理
##目录
1.事件模型-冒泡和捕获
2.事件代理-提高性能的解决方案之一

###1.事件模型
*事件模型分为两种，一种是由Netscape定制的`事件捕获`机制，另一种是由IE定制出来的`事件冒泡`机制。<br/>
`捕获阶段(Capture)`指的是事件从根节点开始，逐级派送到子节点，若节点绑定了事件动作，则执行动作，然后继续走。<br/>
`冒泡阶段(Bubble)`指的是事件从子节点开始，逐级往根节点派送，若节点绑定了事件动作，则执行动作，然后继续走。<br/>
事件执行的顺序：`先捕获，再冒泡`<br/>

由于两种机制由两个厂商定制，所以就有不同的时间绑定方式，其中，遵循标准的浏览器使用W3C定义的`addEventListener`函数绑定，函数定义如下：
`````
function addEventListener(string eventFlag, function eventFunc, [bool useCapture=false])
eventFlag : 事件名称，如click、mouseover…
eventFunc: 绑定到事件中执行的动作
useCapture: 指定是否绑定在捕获阶段，true为是，false为否，默认为true
在事件监听流中可以使用event.stopPropagation()来阻止事件继续往下流

`````
IE中使用自有的attachEvent函数绑定时间，函数定义如下：
``````
function attachEvent(string eventFlag, function eventFunc)
eventFlag: 事件名称，但要加上on，如onclick、onmouseover…
eventFunc: 绑定到事件中执行的动作
在事件监听流中可以使用window.event.cacenlBubble=false来阻止事件继续往下流

```````
还有第三种事件绑定方法：
````
element.on[type]=function(){..}
````
element是绑定事件的元素，type是事件类型，function是绑定事件的执行函数

总结：addEventListener(string eventFlag, function eventFunc, [bool useCapture=false])，针对`ff`，`chrome`，`safari`浏览器，false指冒泡阶段，默认为true，指捕获阶段。不过一般我们都用false。
 attachEvent(string eventFlag, function eventFunc)，针对`ie`系列、还有`opera`浏览器，少了事件处理机制的参数，只指定事件类型（别忘了on）和触发哪个函数。
 而三种函数的兼容性写法应该是:addEventListener>attachEvent>on[type]

好了，有了上面的认知，我们来测试一下
在一个html中，写上
```
<div id="parent">
    <button id="child">子元素事件</button>
</div>
```
给div和button都绑定上事件
```
var parent=document.getElementById("parent"),
            child=document.getElementById("child");

  parent.addEventListener("click",function () {
                 console.log('addEventListener parent div');
  });
   child.addEventListener("click", function (e) {
                   console.log(' addEventListener child btn');
   }) ;
```
点击button结果是

```
addEventListener child btn
addEventListener parent div

```

但我们本不希望div的事件被执行（不希望事件冒泡上去），child那段要改为
```
 child.addEventListener("click", function (e) {
                console.log(' addEventListener child btn');
                var event=e||window.event;
                event.stopPropagation();
            })
```
`event.stopPropagation()`是用来阻止冒泡的，在attachEvent方式下用 `event.cancelBubble = true`;

此时点击button，只会出现
```
addEventListener child btn
```
为了兼容IE和标准浏览器，以上事件绑定过程可以写为：
````
 if(window.addEventListener){
            parent.addEventListener("click",function () {
                console.log('addEventListener parent div');
            })
            child.addEventListener("click", function (e) {
                console.log(' addEventListener child btn');
                var event=e||window.event;
                event.stopPropagation();
            }) ;
        }else if(window.attachEvent){
            parent.attachEvent("onclick", function () {
                console.log('attachEvent parent div');

            });
            child.attachEvent("onclick", function (e) {
                console.log(' attachEvent child btn');
                var event=e||window.event;
                event.cancelBubble=true;
            })
        }else{
            parent.onclick= function () {
            console.log('onclick parent div');
        };
        child.onclick= function () {
            console.log('onclick child btn');
        };
        }
````
如果想把以上事件绑定提取为公用的函数来用，则

````
 var eventUtil={
//            添加事件
            addHandler: function (element,type,handler) {
                if(window.addEventListener){
                    element.addEventListener(type,handler,false);
                }else if(window.attachEvent){
                    element.attachEvent("on"+type,handler);
                }else{
                    element["on"+type]=handler
                }
            },
            //移除事件
            removeHandler: function (element, type, handler) {
                if(window.removeEventListener){
                    element.removeEventListener(type,handler,false);
                }else if(window.detachEvent){
                    element.detachEvent("on"+type,handler);
                }else{
                    element["on"+type]=null
                }
            }
            ,getEvent: function (event) {
                return event?event:window.event;
            }
            //获取事件类型
            ,getType: function (event) {
                return event.type;
            }
            //获得事件对象
            ,getElement: function (event) {
                return event.target||event.srcElement;
            }
            //阻止默认行为
            ,preventDefault: function (event) {
                if(event.preventDefault){
                    event.preventDefault()
                }else{
                    event.returnValue=false;
                }
            }
            //阻止冒泡
            ,stopPropagation: function (event) {
                if(event.stopPropagation){
                    event.stopPropagation()
                }else{
                    event.cancelBubble=true;
                }
            }
        }
````
调用eventUtil的函数即可








