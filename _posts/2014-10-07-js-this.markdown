---
layout: post
---

每条JavaScript代码都是在一个`上下文(context)`中被执行的。

每个`执行上下文(execution contexts)`都划定一个作用域，这些作用域存在于一个`作用域链条(scope chain)`上。

当一个函数被解释执行时，会创建一个新的执行上下文，该上下文所划定的局部作用域同时也会被添加到函数定义所在的作用域链条上。
 
JavaScript在判定一个标识符的时候，会沿作用域链条向上爬，从局部到全局。

所以局部变量优先级高于外层作用域中的变量。

In addition to establishing a scope chain, each execution context offers a keyword named this. 

执行上下文除了创建作用域链条，还提供一个关键字叫做`this`。

这个关键字比较特殊，下面分四种情况讨论。

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

不同于其他普通变量，this的值不是在作用域链条上查询得到的，而是每进入一个执行上下文都会被重设。

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
