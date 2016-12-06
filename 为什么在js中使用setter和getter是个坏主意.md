getters和setters使用起来非常方便，但是它们也可能带来在
运行时不明显的错误。这些问题有什么解决方案？  
如你所知，getter方法和setter方法现在已经成为JavaScript
的一部分。它们在所有主要浏览器中得到广泛支持，甚至从包括IE8以上的IE浏览器。  
一般来说，我不认为这种语法有什么不妥，但是我认为它并不适合JavaScript，
看起来使用getters和setters能够节省时间，也能简化代码，但是它们会带来
隐藏的错误，乍一看上去并不能发现。  
## Getters和Setters如何工作？
先简单回顾一下
> 有的时候需要获取一个属性动态的计算值或者不使用方法调用
获得一个内部变量的状态。  （这么说不是很明白，翻译不了精髓）   

为了说明他们是如何工作的，让我们来看看一个具有两个属
性的`person`对象,`firstName`和`lastName`为它的两个属性
，和一个计算属性：`fullName`。  
```javascript
VAR  OBJ  = {
    firstName: "Maks",
    lastName："Nemisj"
}
```
计算属性会将`firstName`和`lastName`两个字符串连接起来并返回

```javascript
Object.defineProperty(person, 'fullName', {
  get: function () {
    return this.firstName + ' ' + this.lastName;
  }
});
```
要得到这样一个`fullName`属性的值并不需要函数调用的方式
比如`person.fullName()`,只要简单的`var fullName = person.fullName`
就可以了  
同样的setters也是一样，你可以设定一个值通过以下的function：

```javascript
Object.defineProperty(person, 'fullName', {
  set: function (value) {
    var names = value.split(' ');
    this.firstName = names[0];
    this.lastName = names[1];
  }
});
```
使用起来也很简单：`person.fullName = 'Boris Gorbachev'`
设置它的`fullName`属性会被setters拦截，并执行上面定义的函数，将这个
fullName 字符串分开，并存入对应的 `firstName`和`lastName`属性中。
## 哪里有问题？
你也许在想：“嘿，我喜欢getter和setter方法，感觉更自然，就像使用
JSON一样。你说得对，的确这样，但是让我们回头看看`fullName`在getters和
setters之前是怎么工作的。  
要得到一个值我们可能会用到像`getFullName()`这样的方法，同样的，给一个值
赋值`person.setFullName('Maks Nemisj')`会被使用到。  
如果这个函数的名字写错了，
`person.getFullName()`写成了 `person.getFulName()`？会怎么样  
JavaScript会给出一个错误
```javascript
person.getFulName();
       ^
TypeError: undefined is not a function
```
这个错误会在正确的时间正确的地点被触发。尝试着去调用一个对象并
不存在的方法会抛出一个错误-很正确。  
现在让我们看看当使用一个错误的名字来使用setter会怎么样？
```
person.fulName = 'Boris Gorbachev';
```
什么也不会发生，对象是可拓展的，可以动态的赋予键和值，所以在运行
时不会抛出任何错误。  
这样做意味着错误可能会在用户界面的某个地方被发现，或者，在计算一些错误的
值的时候，但不会是这个在拼写错了的时候。  
跟踪这样一个本应该在过去发生的错误，却在未来的代码流里展现出来“很有趣”。
## 使用seal来解决
这个问题可以部分的通过`seal`API来解决。当一个对象被密封（
密封对象是指那些不能添加新的属性，不能删除已有属性，以
及不能修改已有属性的可枚举性、可配置性、可写性，但可能可
以修改已有属性的值的对象。），它就不能被突变，也就意味着
`person.fulName = 'Boris Gorbachev'`这个操作并不能成功。  
由于种种原因，当我在node.js v4.0中测试这些的时候，它们表现的和我想的并不
一样，所以我对这个结论并不确定。  
更令人沮丧的是，几乎没有任何方法能够解决setter的问题。正如我提到的，对象
是可拓展的，在出现问题时会报错的，这意味着在访问不在的属性也不会导致任何错误。  
我不会烦心写这篇文章如果这种解决方法只能应用于对象字面量， 但是随着es6的兴起，
以及在class内写getters和setters变成可能，我决定去写一些可能出现的陷阱。
## 大家对类的态度
我知道现在class在javascript社区并不是很受欢迎。人们在辩论在一个以函数/原型继承
为核心的javascript语言中类是否真的有必要，然而事实上,class已经出现在了es6
的规范中了。  
对我来说，类是指定在类的外部以及应用内部之间定义好的api接口的一种方式。
它 是一个将规则放进黑白片的抽象,假定这些规则在一段时间内不会改变。  
是时候将person对象写成类的形式了，Person类定义了获取和设置fullname的接口

```javascript
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  getFullName() {
    return this.firstName + ' ' + this.lastName;
  }
  setFullName(value) {
    var names = value.split(' ');
    this.firstName = names[0];
    this.lastName = names[1];
  }
}
```
类定义了严格的接口描述,但是getters和setters让它变得不是那么严格，
我们已经习惯了在使用对象字面量和JSON时发生的拼写错误。
至少我希望类会更严格，并为开发者更好的提供反馈意见。  
虽然在类上定义getter和setter时这种情况没有什
么不同。它不会阻止其他人犯打字错误，没有任何反馈。

```javascript
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  get fullName() {
    return this.firstName + ' ' + this.lastName;
  }
  set fullName(value) {
    var names = value.split(' ');
    this.firstName = names[0];
    this.lastName = names[1];
  }
}
```
拼写错误并不会抛出
```javascript
var person = new Person('Maks', 'Nemisj');
console.log(person.fulName);
```



