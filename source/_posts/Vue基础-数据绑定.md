layout: "post"
title: "Vue基础-数据绑定"
date: "2019-04-20 20:47"
categories:
- [UI,VUE]
tags:
- [TECHNOLOGY]
thumbnail: http://swcheng.com/images/vuelogo.png
---
　　在引入Vue需要的js文件之后，比如我们在js代码里创建下面这样一个Vue对象:
<!-- more -->

{% codeblock lang:JavaScript %}
    // 声明一个组件
    Vue.component('my-component', {
      template: '<p class="foo bar">Hi</p>'
    });

    var app = new Vue(
    {
      el: '#app',
      data: {
        message: "Hello World!",
        seen: true,
        dynamicId: "iamid",
        attributeName: "href",
        addre: "http://www.baidu.com",
        isDisable: true,
        vhtml: "<input>",
        firstName: 'Foo',
        lastName: 'Bar',
        fullName: 'Foo Bar'
        todos: [
          {text:  "Learn JavaScript" },
          {text:  "Learn Vue" },
          {text:  "Build something awesome" }
        ],
        groceryList: [
          {
            id: 0,
            text: 'Vegetables'
          },
          {
            id: 1,
            text: 'Cheese'
          },
          {
            id: 2,
            text: 'Whatever else humans are supposed to eat'
          }
        ],
        // class和style属性的绑定
        isActive: true,
        hasError: true,
        classObject: {
          cla1: true,
          cla2: true
        },
        ind1: "class1",
        ind2: "class2",
        activeColor: "red",
        fontSize: 20,
        styleObject: {
          color: 'pink',
          fontSize: '13px'
        }
      },
      methods: {
        reverseMessage: function(){
          this.message = this.message.split('').reverse().join('');
        }
      },
      computed: {
        reversedMessage: function () {
          return this.message.split('').reverse().join('')
        }
      },
      watch: {
        firstName: function (val) {
          this.fullName = val + ' ' + this.lastName
        },
        lastName: function (val) {
          this.fullName = this.firstName + ' ' + val
        }
     }
    });

{% endcodeblock %}
### 文本解析
　　可以使用Mustache标签去做原始的文本解析，这样的解析后的结果不会作为html代码输出:
{% codeblock lang:html %}
    <span>Message: {% raw %}{{ message }}{% endraw %}</span>
{% endcodeblock %}
　　结果显示:
{% codeblock lang:html %}
    <span>Message: Hello World!</span>
{% endcodeblock %}
　　如果不希望span里的内容随着message的属性变化而变化，可以使用一次赋值，下面的messgae的值不会改变:
{% codeblock lang:html %}
    <span v-once>Message: {% raw %}{{ message }}{% endraw %}</span>
{% endcodeblock %}

### Html解析
{% codeblock lang:html %}
    <span v-html="vhtml"></span>
{% endcodeblock %}
　　解析后的结果为:
{% codeblock lang:html %}
    <span>
      <input>
    </span>
{% endcodeblock %}

### 属性解析
　　使用v-bind指令:
{% codeblock lang:html %}
　　<div v-bind:id="dynamicId"></div>
{% endcodeblock %}
　　解析结果:  
{% codeblock lang:html %}
　　<div id="iamid"></div>
{% endcodeblock %}
　　动态绑定对应属性:
{% codeblock lang:html %}
　　<a v-bind:[attributeName]="addre"> ... </a>
{% endcodeblock %}
　　该节点对应的vue对象的attributeName的值将作为将要设置的对应属性。如果是href记得要加{% raw %}http://{% endraw %}前缀，不然会当做ftp。一个需要注意的地方是attributeName在有些浏览器会被全部被当做小写去解析，所以这个值我们尽量可能在vue对象中定义为小写。

　　当然也可以解析除了id之外的其它任何属性。注意以上无论是使用v-指令亦或者是Mustache标签，都可以使用js表达式去获得最终的值，但是仅仅可以使用一个js表达式去计算，而不可以写一堆js语句。

### 计算属性
　　计算属性是我们在vue对象中定义的需要通过一些js代码计算获得最终值的属性，这些属性的变化取决于它需要计算的属性的变化，这个对应的属性被称为响应式依赖。比如这里的message即使需要被计算的对象，当message变化的时候reversedMessgae属性就会发生变化。
{% codeblock lang:html  %}
    <div id="example">
      <p>Computed reversed message: "{% raw %}{{ message }}{% endraw %}"</p>
    </div>
{% endcodeblock %}

　　如果你不需要当响应式依赖改变的时候才可以改变计算属性的值，你想要自己手动直接修改，那么你可以修改为下面。当然这里的cache意味着你访问这个computed的值的时候，它是否会使用之前的缓存值，不使用则是是重新调用get方法。而且，即使我们重写这个set方法，其实作用也不大，因为我们再次获取就会调用get方法，返回不是我们改变的值，所以只有在set中改变对应的响应式依赖，这里的改变才会变得有意义。
{% codeblock lang:js %}
computed: {
  reversedMessage: {
    cache: false,
    // getter
    get: function () {
      return this.message.split('').reverse().join('')
    },
    // setter
    set: function (newValue) {
      this.message = newValue
    }
  }
}
{% endcodeblock %}

### 侦听属性
　　侦听属性，就是我们需要进行监听其值的变化的属性，然后执行我们需要的操作。每当firstName或者lastName的值被修改的时候，对应的watch方法就会被调用。
{% codeblock lang:js  %}
    watch: {
      firstName: function (val) {
        this.fullName = val + ' ' + this.lastName
      },
      lastName: function (val) {
        this.fullName = this.firstName + ' ' + val
      }
    }
{% endcodeblock %}

　　对于class属性以及style属性，如果去做一些字符串拼接和计算，则会特别繁琐，故而vue对这些属性的绑定做了一些特别的处理。

### class属性绑定
　　直接引用vue对象中定义的对象或者手动创建一个对象亦或者是一个属性，或者是使用一个包含对象以及属性的数组:
{% codeblock lang:html  %}
  <!-- 直接在双引号中使用js表达式返回一个对象，如果在vue对象中，isAcitve是true则存在这个class否则就是没有 -->
  <span v-bind:class="{ active: isActive, 'text-danger': hasError}"></span>
  <!-- 直接引用一个对象 -->
  <span v-bind:class="classObject"></span>
  <!-- 传递一个数组，这里是将数组里的属性作为对象中的属性值传递到里面去，这样就不存在是否有true或false这一说法了 -->
  <span v-bind:class="[ind1, ind2]"></span>
  <!-- 也可以将属性和对象共同使用 -->
  <span v-bind:class="[ind1, ind2, classObject]"></span>
  <!-- 如果想要达到使用数组切换的目的，那么可以使用三元表达式 -->
  <span v-bind:class="[isActive ? ind1: '', ind2]"></span>
{% endcodeblock %}

　　解析结果:
{% codeblock lang:html %}
  <span class="active text-danger"></span>
  <span class="cla1 cla2"></span>
  <span class="class1 class2"></span>
  <span class="class1 class2 cla1 cls2"></span>
  <span class="class1 class2"></span>
{% endcodeblock %}　　
　　对于自定义组件，如果在定义中就已经存在一些值，那么会在其后进行追加。
{% codeblock lang:html %}
  <!-- 组件的定义必须放在最前面 -->
  <my-component v-bind:class="{ active: isActive }"></my-component>
{% endcodeblock %}　　  
　　解析结果:
{% codeblock lang:html %}
  <p class="foo bar active">Hi</p>
{% endcodeblock %}　　
　　可以与普通的class属性共存，也就是可以使用class与v-bind共同完成属性的设置。
{% codeblock lang:html %}
  <!-- 也可以与普通的class属性共存 -->
<div class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
{% endcodeblock %}　
　　解析结果:
{% codeblock lang:html %}
  <div class="static active text-danger"></div>
{% endcodeblock %}

### 绑定内联样式
{% codeblock lang:html %}
  <!-- 绑定内联样式，使用对象去绑定，不过定义在vue对象中更加合适 -->
  <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
  <!-- 定义在对象中，当然也可以使用数组来引用多个对象 -->
  <div v-bind:style="styleObject">123</div>    
  <!-- 可以为一些属性添加多重值 -->
  <div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
{% endcodeblock %}

　　解析结果:
{% codeblock lang:html %}
  <div style="color: red; font-size: 20px;"></div>
  <div style="color: pink; font-size: 13px;">123</div>
  <div style="display: flex;"></div>
{% endcodeblock %}

### 命令缩写
{% codeblock lang:html %}
    <!-- 完整语法 -->
    <a v-bind:href="url">...</a>

    <!-- 缩写 -->
    <a :href="url">...</a>

    <!-- 完整语法 -->
    <a v-on:click="doSomething">...</a>

    <!-- 缩写 -->
    <a @click="doSomething">...</a>
{% endcodeblock %}

### 注意
　　在编写这篇Vue笔记的时候，会出现Mustache标签与Nunjucks语法冲突的问题，解决方案可以查看 [这里](https://github.com/hexojs/hexo/issues/1930 "Vue.js 中的双大括号{{ Mustache }}与 Nunjucks 解析相冲突")。



