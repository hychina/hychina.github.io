---
layout: post
---

####上下文和作用域

每条JavaScript代码都是在一个`上下文(context)`中被执行的。每次函数调用都会创建一个新的`执行上下文(execution contexts)`。

JavaScript中的函数实际上都是对象，而每个函数对象都有一个[[scope]]属性，JavaScript是通过这个属性来访问`作用域链条(scope chain)`的。

当我们创建一个函数对象时(即定义一个函数时)，函数对象内部的[[scope]]属性会**指向函数定义所在上下文的作用域链条**。而当这个函数被调用执行时，会创建一个新的执行上下文，这个上下文代表一个局部作用域，这个局部作用域也会被添加到[[scope]]所指向的作用域链条上。

例如，如果在全局上下文中定义一个函数，那么这个函数对象的[[scope]]属性会指向全局上下文的作用域链条，而这个作用域链条只包含一个全局对象。

{% highlight javascript %}
function exampleFunction(formalParameter){
    ...   // function body code
}
{% endhighlight %}

而如果在一个函数内部定义另一个函数，那么内部函数就被定义在了外部函数的执行上下文中。
 
{% highlight javascript %}
function exampleOuterFunction(formalParameter){
    function exampleInnerFuncitonDec(){
        ... // inner function body
    }
    ...  // the rest of the outer function body.
}

exampleOuterFunction( 5 );
{% endhighlight %}

这里exampleOuterFunction定义在全局上下文中，所以其[[scope]]属性指向仅包含全局对象的作用域链条。 而当exampleOuterFunction被执行时，会创建一个新的执行上下文，这个执行上下文的作用域包含了局部作用域以及exampleOuterFunction的[[scope]]属性所指向的作用域链条(仅包含全局对象)。内部函数exampleInnerFunctionDec的[[scope]]属性指向的作用域链条和这个执行上下文的作用域相同，包含了exampleOuterFunction的局部作用域以及全局作用域。

通过with表达式，我们能够改变作用域。

{% highlight javascript %}
// 创建全局变量y，指向一个对象
var y = {x:5}; // object literal with an - x - property

function exampleFuncWith(){
    var z;

    // with语句将全局变量y所指向的对象添加到作用域链条的前端(在局部作用域之前)
    with(y){
        z = function(){
            ... // inner function expression body;
        }
    }
    ... 
}

exampleFuncWith();
{% endhighlight %}

with语句结束之后，作用域链条会回复原状，不过with语句内定义的函数其[[scope]]属性中已经多了一个全局对象y，并且y在链条的最前端。

##标识符判定##

JavaScript在判定一个标识符的时候，会沿作用域链条向上爬，局部作用域在全局作用域之前，所以局部变量优先级高于外层作用域中的变量。

标识符的判定从作用域链条上的第一个对象开始。JavaScript检查链条上的对象是否有一个属性和所要判定的标识符相同(因为要检查对象的属性，如果对象有原型链的话也会沿原型链进行检查)。如果第一个对象没有对应属性，那么再检查链条上的下一个对象，直到找到一个包含对应属性的对象或者穷尽所有对象为止。

####闭包

一般来说，在退出一个执行上下文之后，该执行上下文中的对象和函数对象就无法在外部访问了，所以可以对其进行垃圾回收。 但闭包能避免这一点。要构造一个闭包，可以在退出一个执行上下文时，将内部定义的一个函数对象作为返回值返回，或将其赋给一个全局变量，或者一个全局对象或参数对象的某个属性。

{% highlight javascript %}
function exampleClosureForm(arg1, arg2){
    var localVar = 8;
    function exampleReturned(innerArg){
        return ((arg1 + arg2)/(innerArg + localVar));
    }
    // 返回内部定义的函数对象
    return exampleReturned;
}

var globalVar = exampleClosureForm(2, 4);
{% endhighlight %}

现在内部函数exampleReturned就不会被垃圾回收了，因为他被赋给了全局变量globalVar，所以仍然可以访问到。 而更重要的是由于[[scope]]属性的存在，exampleReturned函数的作用域链条上的所有对象都不会被垃圾回收了。这样我们在外部就仍能访问到localVar、arg1以及arg2了。

####this关键字

执行上下文除了创建作用域链条，还提供一个关键字叫做`this`。 ##不同于其他普通变量，this的值不是通过作用域链条得到的，而是每进入一个执行上下文都会被重设。## 这个关键字比较特殊，下面分四种情况讨论。

**调用一个对象的方法时**

此时this和传统oop语言中的this并无区别。可以用this来访问翠香本身的属性和方法。

{% highlight javascript %}
var obj = { 
   number: 42, 
   getNumber: function () { 
     return this.number; 
   } 
 }; 
{% endhighlight %}

var number = obj.getNumber(); 

当obj.getNumber()被执行时，JavaScript会为这个函数调用创建一个执行上下文，并将this设为圆点前面的对象，即obj。这样对象就能借助this访问自身属性了。

**构造函数**

当使用new关键字来调用构造函数的时候，this指向的是所创建的对象。

{% highlight javascript %}
function Object(number) { 
  this.number = number; 
  this.getNumber = function () { 
    return this.number; // 注意这里的this和外面的this不同
  } 
} 
var obj = new Object(42); 
var number = obj.getNumber(); 
{% endhighlight %}
  
注意getNumber函数中的this和Object构造函数中的this是不同的。 我们是通过new关键字来执行Object构造函数的，所以此时this代表正在被创建的新对象。 而另一方面，我是通过obj对象来调用getNumber函数的，所以函数被执行时，this代表obj对象。

**函数调用**

如果不牵扯对象，只是调用一个普通函数，此时this代表什么？

{% highlight javascript %}
function test_this() { 
  return this; 
} 
var i_wonder_what_this_is = test_this(); 
{% endhighlight %}

此时this默认指向最外层的全局对象；对网页来说指向的是window对象。

**事件处理器**

假设我们用一个函数来处理onclick事件，当事件触发导致函数被调用时，this指向什么？

这个问题比较复杂。

{% highlight javascript %}
<script type="text/javascript"> 
  function click_handler() { 
    alert(this); // alerts the window object 
  } 
</script> 
 ... 
<button id='thebutton' onclick='click_handler()'>Click me!</button>
{% endhighlight %}

如果用上面这种写法，this指向的是全局的window对象。

如果事件处理器是通过JavaScript添加的，那么this指向的是产生事件的那个DOM元素。

{% highlight javascript %}
<script type="text/javascript"> 
  function click_handler() { 
    alert(this); // alerts the button DOM node 
  } 
  
  function addhandler() { 
    document.getElementById('thebutton').onclick = click_handler; 
  } 
  
  window.onload = addhandler; 
</script> 
 ... 
<button id='thebutton'>Click me!</button>
{% endhighlight %}

如果把上面例子改成下面这样：

{% highlight javascript %}
<script type="text/javascript"> 
  function Object(number) { 
    this.number = number; 
    this.getNumber = function () { 
      alert(this.number); 
    } 
  } 
  
  function addhandler() { 
    var obj = new Object(42), 
    the_button = document.getElementById('thebutton'); 
  
    the_button.onclick = obj.getNumber; 
  } 
  
  window.onload = addhandler; 
</script>
{% endhighlight %}

如果运行上面这段代码，我们得到的不是42，而是"undefined".

原因是函数的执行上下文变了。我们之前在调用obj.getNumber()时，this指向的是obj对象。而现在我们并没有通过obj来调用getNumber函数，我们只是将getNumber作为`回调函数(callback)`传递给了the_button对象。所以当事件触发onclick被调用时，实际上是通过the_button对象调用了getNumber函数，此时this指向的是DOM元素the_button，而这个对象并没有number这个属性，所以最终输出的是"undefined"。

setTimeout函数也有相同的效果，不但延迟一个函数的执行，同时也将其放入全局上下文中。

`window.setTimeout("javascript function", milliseconds);`

**通过apply()和call()来掌控上下文**

当执行一个函数调用时，apply和call能够让我们手动地覆盖this的默认值。

{% highlight javascript %}
<script type="text/javascript"> 
  var first_object = { 
    num: 42 
  }; 
  var second_object = { 
    num: 24 
  }; 
  
  function multiply(mult) { 
    return this.num * mult; 
  } 
  
  multiply.call(first_object, 5); // returns 42 * 5 
  multiply.call(second_object, 5); // returns 24 * 5 
</script>
{% endhighlight %}

call的第一个参数指定了在函数执行时this代表的是什么。(有些语言中this会作为隐含参数传递给函数，这里相当于将this参数显式地写出来了。)

apply与call的工作原理相同，只不过允许我们将参数放在一个数组中。

{% highlight javascript %}
<script type="text/javascript"> 
 ... 
  
  multiply.apply(first_object, [5]); // returns 42 * 5 
  multiply.apply(second_object, [5]); // returns 24 * 5 
</script>
{% endhighlight %}

现在你也许会认为下面这段代码能解决之前遇到的执行上下文转换问题。

{% highlight javascript %}
function addhandler() { 
  var obj = new Object(42), 
  the_button = document.getElementById('thebutton'); 
  
  the_button.onclick = obj.getNumber.call(deep_thought); 
}
{% endhighlight %}

这段代码问题明显，我们没有将getNumber传递给the_button，而是立即执行了这个函数，将结果赋给了onclick。
下面介绍真正的解决方案。

**bind()的美妙**

和call一样，bind的作用也是指定函数执行时的上下文。不同时bind不会立即执行一个函数，而是返回这个函数的一个引用。

下面通过代码演示bind的工作原理。

{% highlight javascript %}
<script type="text/javascript"> 
  var first_object = { 
    num: 42 
  }; 
  var second_object = { 
    num: 24 
  }; 
  
  function multiply(mult) { 
    return this.num * mult; 
  } 
  
  Function.prototype.bind = function(obj) { 
    var method = this, 
    temp = function() { 
      return method.apply(obj, arguments); 
    }; 
  
    return temp; 
  } 
  
  var first_multiply = multiply.bind(first_object); 
  first_multiply(5); // returns 42 * 5 
  
  var second_multiply = multiply.bind(second_object); 
  second_multiply(5); // returns 24 * 5 
</script>
{% endhighlight %}

Function.prototype.bind使得所有函数都具有了bind这个方法。
当multiply.bind被调用时，JavaScript为bind方法创建一个执行上下文，并将this设置为multiply函数。

这都没有问题，关键在于method这个变量。

method变量记录了bind执行时this的值，而在下一行创建的匿名函数中，method的值是通过作用域链条得到的(而不是像this那样每次调用都被重置)，obj的值同样如此。


这里的temp是一个闭包(`closure`)，能够记录

现在，我们可以用下面的方式解决之前的问题了。

{% highlight javascript %}
function addhandler() { 
  var obj = new Object(42), 
  the_button = document.getElementById('thebutton'); 
  
  the_button.onclick = obj.getNumber.bind(obj); 
}
{% endhighlight %}
