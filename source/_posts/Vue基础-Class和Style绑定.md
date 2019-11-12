layout: "post"
title: "Vue基础 - Class和Style绑定"
date: "2019-10-28 10:00"
categories:
- [UI,VUE]
tags:
- [TECHNOLOGY]
thumbnail: http://swcheng.com/images/vuelogo.png
---
　　在开发中我们不可避免的要动态的改变一个元素的样式，有时候是为了响应用户操作，有时候是根据数据的不同显示不同的样式等。这些时候我们的样式不可以进行硬编码，在正常情况下我们需要手动编写JS去实现这个功能，但是在Vue组件中，可以使用数据绑定一样的方式为组件的属性进行赋值达到动态修改组件样式的目的，由于Vue强大的响应式功能，我们可以很方便的修改组件的class和style属性。

<!-- more -->

　　Vue对class和style两个属性做了特别的增强，一般的属性的值只能为js计算结果的字符串，但class和style除了字符串外，还可以是对象或者数组：

## class绑定

### 使用对象的方式

　　为class属性添加一个对象，

{% codeblock lang:html %}
　　<div v-bind:class="{ active: isActive }"></div>
{% endcodeblock %}

　　在使用对象表达一个组件的class的时候，对象的属性表示class的值，而该属性的值isActive是否为一个真值，即true或false，如果不是true或false则会被强制转化为true或false值。当其为true的时候表示改组件有这个属性表示的class，否则没有，所以你可以通过在这个对象中添加多个属性达到控制组件class的目的。比如使用下面的方式：

{% codeblock lang:html %}
  <div
    v-bind:class="{ active: isActive, 'text-danger': hasError }">
  </div>
{% endcodeblock %}

　　此外，v-bind:class属性还可以和普通的class属性共存，比如有下面的例子：

{% codeblock lang:html %}
  <div
    class="static"
    v-bind:class="{ active: isActive, 'text-danger': hasError }">
  </div>
{% endcodeblock %}

　　如果有以下data，

{% codeblock lang:js %}
  data: {
    isActive: true,
    hasError: false
  }
{% endcodeblock %}

　　结果会被渲染为，

{% codeblock lang:html %}
  <div class="static active"></div>
{% endcodeblock %}

　　当isActive或者hasError的值变化时，class列表将相应地更新。例如，如果hasError的值为true，class列表将变为"static active text-danger"。当然这里前提是isActive和hasError需要满足响应式的前提，由于js的限制，vue无法对对象属性的增加和删除进行响应，也没法对数组的非变异方法进行响应。

### 使用数组的方式

　　我们可以把一个数组传给 v-bind:class，

{% codeblock lang:html %}
  <div v-bind:class="[{ active: isActive }, errorClass]"></div>

  data: {
    errorClass: 'text-danger',
    isActive: true
  }
{% endcodeblock %}

　　与对象的渲染方式不同，使用数组会判断在数组中的变量类型，如果变量的值的类型为一个对象，那么它会像传递对象的方式去处理这个变量，如果这个变量的值的类型为一个字符串，那么会将其代表的字符串的值作为组件的class属性，

{% codeblock lang:html %}
  <div class="active text-danger"></div>
{% endcodeblock %}

　　在使用组件的方式的时候，你可以使用三元表达式去控制数组中的成员达到控制组件属性的目的：
{% codeblock lang:html %}
  <div v-bind:class="[{ active: isActive }, errorClass]"></div>
{% endcodeblock %}

### 与模版class共存

　　之前我们已经知道v-bind:class可以与class共存，除此之外，其实v-bind:class以及class都同时可以和组件定义处的属性共存。例如，你声明了下面的组件，
{% codeblock lang:html %}
  Vue.component('my-component', {
    template: '<p class="foo bar">Hi</p>'
  })
{% endcodeblock %}

　　如果直接添加一些class，
{% codeblock lang:html %}
  <my-component class="baz boo"></my-component>
{% endcodeblock %}

　　结果会被渲染为：
{% codeblock lang:html %}
  <p class="foo bar baz boo">Hi</p>
{% endcodeblock %}

　　同样对于v-bind:class也同样适用，
{% codeblock lang:html %}
  <my-component v-bind:class="{ active: isActive }"></my-component>
{% endcodeblock %}

　　当 isActive 为 truthy[1] 时，HTML 将被渲染成为：
{% codeblock lang:html %}
  <p class="foo bar active">Hi</p>
{% endcodeblock %}


## style内联样式绑定

### 使用对象的方式

　　与class绑定有所区别，在内联样式的绑定中，对象的属性和值分别表示css属性的名称和值，需要注意的是，对象属性的值类型必须为字符串而不可以是其它任意类型的值，因为vue无法判断对象属性中的值的单位，所以我们必须使用字符串进行准确的定义。CSS的很多属性都使用横线分割表示，我们在js中对象定义属性的时候无法直接定义短横线分割的属性，虽然可以通过加单引号包含的方式添加这样的属性，但是访问起来也必须通过[]的方式引用，这样未免太不人性化，所以vue提供了驼峰式(camelCase)的方式在对象中定义属性达到定义css属性的方式：

{% codeblock lang:html %}
  <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

  data: {
    activeColor: 'red',
    fontSize: 30
  }
{% endcodeblock %}

　　直接绑定到一个样式对象通常更好，这会让模板更清晰：
{% codeblock lang:html %}
  <div v-bind:style="styleObject"></div>

  data: {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
{% endcodeblock %}

　　结果会被渲染为:
{% codeblock lang:html %}
<div style="color: 'red'; fontSize: '13px'"></div>
{% endcodeblock %}
### 使用数组的方式

　　v-bind:style使用数组的方式类似于v-bind:class，可以同时定义多个变量，也可以使用三元表达式，同时根据变量的类型去定义css属性。

### 自动添加前缀

　　在v-bind:style使用一些需要自动添加[]浏览器引擎前缀](https://developer.mozilla.org/zh-CN/docs/Glossary/Vendor_Prefix)的css属性的时候，Vue.js 会自动侦测并添加相应的前缀。

### 多重值

　　从 2.3.0 起你可以为 style 绑定中的属性提供一个包含多个值的数组，常用于提供多个带前缀的值，例如：
{% codeblock lang:html %}
  <div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
{% endcodeblock %}

　　这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 display: flex。
