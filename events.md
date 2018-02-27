# HTML事件相关

## 事件传播
捕获 -> 目标 -> 冒泡
- 事件捕获: 事件发生时首先作用在document上, 然后依次传递给body及子节点, 最后到达目标节点; 在这个阶段阻止事件传播叫"捕获"
- 事件冒泡: 到达目标节点后事件会逐层向上冒泡, 直至document对象, 与事件捕获相反

## API
- onclick
`on+eventType`直接赋值给dom节点, 重新onclick会覆盖之前的属性

阻止默认事件: `return false`

- addEventListener(eventType, handler, boolean)
可以绑定多个方法不被覆盖; 第三个参数false表示在冒泡阶段触发handler处理, true表示在捕获阶段触发; ie8以下不支持

`removeEventListener`与之相对应, 解绑事件处理函数

`event.preventDefault()`阻止通过addEventListener添加的事件的默认事件

`event.stopPropagation()`阻止事件之后的进一步传播, 包括冒泡, 捕获

- attachEvent('on+eventType', handler)
ie特有, 只支持冒泡阶段触发

detachEvent解绑事件

`event.returnValue = false`阻止attachEvent添加的事件的默认事件

`event.cancelBubble = true`阻止进一步冒泡

## 事件委托（代理）
利用事件冒泡的特性，将里层的事件委托给外层事件，根据event对象的属性进行事件委托，改善性能。

使用事件委托能够避免对特定的每个节点添加事件监听器；事件监听器是被添加到它们的父元素上。事件监听器会分析从子元素冒泡上来的事件，找到是哪个子元素的事件。

原生js事件代理:
```javascript

// ============ 简单的事件委托
function delegateEvent(interfaceEle, selector, type, fn) {

    if(interfaceEle.addEventListener){
    interfaceEle.addEventListener(type, eventfn);
    }else{
    interfaceEle.attachEvent("on"+type, eventfn);
    }
     
    function eventfn(e){
    var e = e || window.event;    
    var target = e.target || e.srcElement;
    //如果目标元素与选择器匹配则执行函数
    if (matchSelector(target, selector)) {
            if(fn) {
 //将fn内部的this指向target（在此之前this都是指向的绑定事件的元素即interfaceEle）
                fn.call(target, e); 
            }
        }
    }
}
/**
 * only support #id, tagName, .className
 * and it's simple single, no combination
 */
//比较函数：判断事件的作用目标是否与选择器匹配；匹配则返回true
function matchSelector(ele, selector) {
    // 如果选择器为ID
    if (selector.charAt(0) === "#") {            
        return ele.id === selector.slice(1);   
    }
      //charAt(0),返回索引为0的字符
    //slice(a，b),从已有的数组或字符串返回从索引从a处开始，截取到索引b之前的子数组或子字符串；
    //如果选择器为Class
    if (selector.charAt(0) === ".") {
        return (" " + ele.className + " ").indexOf(" " + selector.slice(1) + " ") != -1;
    }
    // 如果选择器为tagName
    return ele.tagName.toLowerCase() === selector.toLowerCase();
}
//toLowerCase()将字符串转换成小写
//调用
var odiv = document.getElementById("oDiv");
delegateEvent(odiv,"a","click",function(){
    alert("1");
})
```

## [自定义事件创建及触发](http://www.cnblogs.com/stephenykk/p/4861420.html)

- document.createEvent
- document.dispatchEvent
