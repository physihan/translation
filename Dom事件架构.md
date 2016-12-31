# Dom事件架构
[原文](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)
<hr/>
##事件调度和DOM事件流
这一节简单的概述了事件调度的机制，并且描述了事件是怎么通过DOM树传播的。应用可以通过`disapatchEvent()`方法来调度事件对象，事件对象会根据DOM事件流来传播。  

![tupian](./asset/img/dom.svg) 