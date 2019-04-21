layout: "post"
title: "Vue基础-事件处理和表单数据绑定"
date: "2019-04-21 18:56"
categories:
- [UI,VUE]
tags:
- [TECHNOLOGY]
thumbnail: http://swcheng.com/images/vuelogo.png
---
　　创建一个vue对象:
<!-- more -->
{% codeblock lang:js %}
  var example1 = new Vue({
    el: '#example-1',
    data: {
      counter: 0,
      message: "",
      multiMessage: "",
      checked: false,
      checkedNames: [],
      picked: "One",
      selected: "A"
    },
    methods: {
      greet: function(event){
        alert(event)
        alert("Greet from vue")
      },
      say: function(message){
        alert(message)
      },
      warn: function(msg, event){
        // 这里我们可以直接调用原生对象event
        if(event) event.preventDefault()
        alert(msg)
      },
      doThis: function(){
        console.log("dothis")
      },
      onSubmit: function(){
        console.log("onSubmit")
      },
      doThat: function(){
        console.log("dothat")
      },
      submit: function(){
        console.log("submit")
      }
    }
  })
{% endcodeblock %}

### 监听事件
　　使用v-on指令去进行基础事件的监听，在触发事件时执行一些js代码:
{% codeblock lang:html %}
  <div id="example-1">
    <!-- 基本的事件处理 -->
    <button v-on:click="counter += 1">Add 1</button>
    <p>The button above has been clicked {% raw %}{{ counter }}{% endraw %} times.</p>
  </div>  
{% endcodeblock %}
　　这里我们监听的是click的事件，每次点击都会使下面的文字显示增加一次点击的次数，但是一般点击事件不应该直接在html里进行处理，我们应该尽可能的将其写在js代码中。比如:
{% codeblock lang:html %}
　<button v-on:click="greet">Greet</button>
{% endcodeblock %}
　　点击这个按钮则会调用greet方法，这里我们直接绑定的是在vue对象中定义的方法名而不带任何参数，当然你也可以选择在这里选择直接调用一个带参数的方法。这里兼顾了属性定义监听和使用DOM定义绑定方法对象两种方式。比如:
{% codeblock lang:html %}
  <button v-on:click="say(message)">Say</button>
{% endcodeblock %}
　　如果我们需要获取原生的DOM对象，可以使用特殊变量$event将其传递到方法中:
{% codeblock lang:html %}
  <button v-on:click="warn('Form cannot be submitted yet.', $event)">
    Submit
  </button>
{% endcodeblock %}
　　对于很多事件，我们很多时候不仅仅是需要一个回调方法，我们很多时候还有其它的操作，对于一些常见的设置，为了方便vue提供了事件修饰符，比如:
- .stop
- .prevent
- .capture
- .self
- .once
- .passive

　　下面是对这些修饰符的解释。
{% codeblock lang:html %}
  <!-- 点击事件将只会触发一次 2.1.4 新增 -->
  <a v-on:click.once="doThis"></a>

  <!-- 阻止单击事件继续传播,相当于调用stopPropogation -->
  <a v-on:click.stop="doThis"></a>

  <!-- 提交事件不再重载页面，相当于阻止了浏览器默认行为，调用了preventDefault() -->
  <form v-on:submit.prevent="onSubmit"></form>

  <!-- 修饰符可以串联 -->
  <a v-on:click.stop.prevent="doThat"></a>

  <!-- 甚至可以只有修饰符 -->
  <form v-on:submit.prevent></form>

  <!-- 添加事件监听器时使用事件捕获模式，这里捕获的是在capture阶段，而非默认的bubble阶段 -->
  <div v-on:click.capture="doThis">...</div>

  <!-- 只当在 event.target 是当前元素自身时触发处理函数,相当于在event的回调方法中加了判断 -->
  <!-- 比如判断this === event.target 决定是否执行代码 -->
  <div v-on:click.self="doThat">...</div>

  <!-- 浏览器滚动事件的默认行为 (即滚动行为) 将会立即触发，为了提升浏览性能，尤其是移动端-->
  <!-- 而不会等待 `onScroll` 完成  -->
  <!-- 这其中包含 `event.preventDefault()` 的情况,与一般的实现有所区别  2.3.0 新增 -->
  <!-- 不可以将.prevet与.passive一起用，因为它们本身就是冲突，如果你这样做了，.prevent会被忽略
    且同时获得一个警告 -->
  <div v-on:scroll.passive="onScroll">...</div>
{% endcodeblock %}
　　很多时候我们需要监听特定按钮的键盘事件，那么此时我们可以去使用一些按键修饰符。比如:
{% codeblock lang:html %}
  <!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
  <input v-on:keyup.enter="submit">
{% endcodeblock %}
　　可以直接将 [KeyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)暴露的任意有效按键名转换为 kebab-case 来作为修饰符。也可以使用按键码去作为修饰符，这keycode已经被废弃了，所以这里也不涉及到该部分的内容。同时会有一些系统修饰键比如ctrl、alt之类的，如果需要触发该类事件可以使用连续的修饰符。这些系统修饰键有:
- .ctrl
- .alt
- .shift
- .meta

　　比如:
{% codeblock lang:html %}
  <!-- Alt + C -->
  <input @keyup.alt.67="clear">

  <!-- Ctrl + Click -->
  <div @click.ctrl="doSomething">Do something</div>
{% endcodeblock %}
　　
### 表单数据绑定
　　在一个表单中只能包含一些表单元素，常见的表单元素有input、textarea、select等。所谓的表单数据绑定在vue中指的是将vue对象中的属性与表单元素的输入进行绑定，也就是说无论是vue对象中属性值的变化或者是表单元素的输入，都会同步修改对方的值。在vue中使用v-model指令去实现这一功能，v-model指令本质也是通过监听用户的输入事件去更新输入。v-model为不同的元素的不同属性去抛出不同的事件，text 和 textarea 元素使用 value 属性和 input 事件；checkbox 和 radio 使用 checked 属性和 change 事件；select 字段将 value 作为 prop 并将 change 作为事件。
　　下面是一些基本组件的使用的例子:
{% codeblock lang:html %}
  <!-- 普通input输入框 -->
  <input v-model="message" placeholder="edit me">
  <p>Message is: {% raw %}{{ message }}{% endraw %}</p>

  <!-- 多行输入 -->
  <span>Multiline message is:</span>
  <p style="white-space: pre-line;">{% raw %}{{ multiMessage }}{% endraw %}</p>
  <textarea v-model="multiMessage" placeholder="add multiple lines"></textarea>

  <!-- 单个复选框 -->
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{% raw %}{{ checked }}{% endraw %}</label>

  <!-- 多个复选框，绑定到同一个数组 -->
  <div id='example-3'>
    <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
    <label for="jack">Jack</label>
    <input type="checkbox" id="john" value="John" v-model="checkedNames">
    <label for="john">John</label>
    <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
    <label for="mike">Mike</label>
    <br>
    <span>Checked names: {% raw %}{{ checkedNames }}{% endraw %}</span>
  </div>

  <!-- 单选按钮  -->
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {% raw %}{{ picked }}{% endraw %}</span>

  <!-- 选择框 -->
  <select v-model="selected">
    <option  value="">请选择</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {% raw %}{{ selected }}{% endraw %}</span>
{% endcodeblock %}














