layout: "post"
title: "Vue基础 - 组件"
date: "2019-10-28 13:23"
categories:
- [UI,VUE]
tags:
- [TECHNOLOGY]
thumbnail: http://swcheng.com/images/vuelogo.png
---
　　组件是Vue设计的核心思想，Vue应用也是由一个个组件组成的，组件可以被复用，也可以根据情况动态显示不同的组件。在Vue的组件中，主要包括template、script以及style三个内容，template是编写的组件的html模版(也可以使用JSX)，里面包含的是页面的基础的html内容，可以使用Vue指令去进行一些额外逻辑的处理；script包含Vue组件内部的一些数据，方法，生命周期钩子等；style中是为组件编写的一些css样式，如果在style中加上scope表示这些样式作用域为组件内，如果没有则表示定义的全局样式。

<!-- more -->

## 组件注册

　　Vue的组件需要定义且注册之后才可以与其它组件配合工作，上面叙述的是组件在.vue模板里的书写方式，如果我们直接在JS中全局定义并注册一个Vue组件的方式为：

{% codeblock lang:js %}
  // 定义一个名为 button-counter 的新组件
  Vue.component('button-counter', {
    data: function () {
      return {
        count: 0
      }
    },
    template: '<button v-on:click="count++">You clicked me {% raw %}{{ count }}{% endraw %} times.</button>'
  })
{% endcodeblock %}

　　当一个组件完成注册之后，就可以被使用在其它组件之中了，全局注册的组件可以在整个Vue应用中使用，比如我们可以在html模板中这样定义，

{% codeblock lang:html %}
  <div id="components-demo">
    <button-counter></button-counter>
  </div>
{% endcodeblock %}

　　所有Vue应用仅有一个根实例，根实例与其它组件的区别是它具备一个el属性，这个属性在其它组件中并不存在，el属性用于将vue应用绑定到唯一的DOM节点上。对于一个已经定义并全局注册的组件，你可以在任意一个地方进行复用，

{% codeblock lang:html %}
  <div id="components-demo">
    <button-counter></button-counter>
    <button-counter></button-counter>
    <button-counter></button-counter>
  </div>
{% endcodeblock %}

　　在这里的button-counter组件中，每个组件都维护自己的数据独立工作，当这份数据是从外部传入的时候，这个规则可能不再成立。相同的组件只可能因为数据不同而显示不同的样式，当依赖式数据改动的时候，视图将会被相应更新。所以为了维护每个组件的独立，在组件内部定义data属性的时候，必须使用方法返回一个对象而不能直接定义为一个对象，正确添加data的方式应该是这样：

{% codeblock lang:js %}
  data: function () {
    return {
      count: 0
    }
  }
{% endcodeblock %}

　　全局注册过的组件在任何地方都可用，但是有时候我们不需要去全局定义一个组件，因为对于很多使用频率较低组件来说，并不需要进行全局注册，只进行局部注册即可，过多的全局注册会导致用户增加下载的JS的体积。对于局部注册的组件，只可以在引入其的组件中使用，不可以在其它组件的模板中使用。局部定义一个组件的方式很简单，在JS中直接定义的方式为：

{% codeblock lang:js %}
  var ComponentA = { /* ... */ }
  var ComponentB = { /* ... */ }
  var ComponentC = { /* ... */ }
{% endcodeblock %}

　　在需要引入其的组件中的components属性中，添加指向这个对象的引用，(局部注册的组件只可以在它自己的模板中使用，不可使用在其子组件中，也就是说，这里同时引入A和B，无法在B的模板中使用A，要想在B中使用A，必须在B中单独引入)

{% codeblock lang:js %}
  new Vue({
    el: '#app',
    components: {
      'component-a': ComponentA,
      'component-b': ComponentB
    }
  })
{% endcodeblock %}

　　或者如果你通过 Babel 和 webpack 使用 ES2015 模块，那么代码看起来更像：

{% codeblock lang:js %}
  import ComponentA from './ComponentA.vue'

  export default {
    components: {
      ComponentA
    },
    // ...
  }
{% endcodeblock %}

　　在引入局部组件之后，在模板中使用只需要引入其在components中编写的组件属性名称，比如这里定义的属性名称为ComponentA，那么在模板中就可以编写<ComponentA></ComponentA>，这样看上去不太符合标准的html标签的写法，所幸的是，Vue中两种写法都支持，当你使用PascalCase的写法的时候，你依然可以在模板中使用<component-a></component-a>这样的写法。但如果你本身就在components中使用的是kebab-case这样的写法，那么在模板中你也只能在这里面使用kebab-case这样的写法。

### 基础组件全局注册

　　对于一些基础组件，这类组件相对比较通用，如果一个个去导入这些组件，那么会导致在很多组件中有大量包含基础组件的长列表，这时候我们想要在全局注册这些组件。如果你使用webpack这类前端打包工具，那么你可以使用[require.context](https://webpack.js.org/guides/dependency-management/)很方便的一次引入大量的基础组件，

{% codeblock lang:js %}
  import Vue from 'vue'
  import upperFirst from 'lodash/upperFirst'
  import camelCase from 'lodash/camelCase'

  const requireComponent = require.context(
    // 其组件目录的相对路径
    './components',
    // 是否查询其子目录
    false,
    // 匹配基础组件文件名的正则表达式
    /Base[A-Z]\w+\.(vue|js)$/
  )

  requireComponent.keys().forEach(fileName => {
    // 获取组件配置
    const componentConfig = requireComponent(fileName)

    // 获取组件的 PascalCase 命名
    const componentName = upperFirst(
      camelCase(
        // 获取和目录深度无关的文件名
        fileName
          .split('/')
          .pop()
          .replace(/\.\w+$/, '')
      )
    )

    // 全局注册组件
    Vue.component(
      componentName,
        // 如果这个组件选项是通过 `export default` 导出的，
        // 那么就会优先使用 `.default`，
        // 否则回退到使用模块的根。
        componentConfig.default || componentConfig
    )

  });

{% endcodeblock %}  


## 组件Prop

### Prop定义和传递

　　组件中的数据分成两个部分，一部分是由父组件传递给自己的，就是这里说的Prop，另一部分是自身的数据，也就是data属性定义的部分。在组件中定义一个Prop最简单的方式，

{% codeblock lang:js %}
  Vue.component('blog-post', {
    props: ['title'],
    template: '<h3>{% raw %}{{ title }}{% endraw %}</h3>'
  })
{% endcodeblock %}  

　　最简单的在父组件中为其赋值的方式，

{% codeblock lang:js %}
  <blog-post title="My journey with Vue"></blog-post>
  <blog-post title="Blogging with Vue"></blog-post>
  <blog-post title="Why Vue is so fun"></blog-post>
{% endcodeblock %}  

　　一个组件可以拥有任意数量的Prop，在组件中访问Prop的属性的方式和data属性一样，这里的title不仅可以是字符串，可以是任何类型的值(对象，数组，布尔值...)。更方便的是，可以配合使用v-for去遍历生成一个子组件列表，

{% codeblock lang:html %}
  <blog-post
    v-for="post in posts"
    v-bind:key="post.id"
    v-bind:title="post.title"
  ></blog-post>
{% endcodeblock %}  

　　一个需要注意的地方是，在HTML中DOM是不区分大小写的，也就是说浏览器会把所有大写字母解释为小写字母，如果使用DOM中的模板，如果需要对camel-case格式的prop赋值必须在模板中用与其等价的kebab-case命名。

{% codeblock lang:html %}
  Vue.component('blog-post', {
    // 在 JavaScript 中是 camelCase 的
    props: ['postTitle'],
    template: '<h3>{% raw %}{{ postTitle }}{% endraw %}</h3>'
  })

  <!-- 在 HTML 中是 kebab-case 的 -->
  <blog-post post-title="hello!"></blog-post>
{% endcodeblock %}  

　　但是如果使用字符串模板，那么这条限制就不存在了，那么什么是字符串模板什么是DOM模板呢？DOM模板是指能被浏览器解析的模板，DOM模板和元素的html混合在一起进行定义，比如下面的例子，id为demo的div既是位于html中，又作为vue的一个组件被定义，这就被称为DOM模板，

{% codeblock lang:html %}
  <body>
    <!-- html模板 -->
    <div id="demo" title="i love jack">
      <span :customId="id">{% raw %}{{message}}{% endraw %}</span>
    </div>
    <script>
      let obj = {
        message: 'hello,world',
        id: 'JS脚本模板'
      }
      var vm = new Vue({
        el: '#demo',
        data: obj,
        prop: ['title']
      })
    </script>
  </body>
{% endcodeblock %}  

　　字符串模板是定义在js代码中用字符串包裹起来进行定义的vue组件，这种方式被称为字符串模板，比如下面的全局注册和局部组件定义都是字符串模板的使用方式，***(注意，通过字符串模板定义的元素，会替换挂载的元素)***

{% codeblock lang:html %}
  <body>
    <div id="template"></div>
    <script type="x-template" id="optioncompTemp">
           <option>a</option>
    </script>
    <script>
      Vue.component('my-component', {
        props: ['param'],
        template: `
          <div>A custom component{% raw %}{{param}}{% endraw %}</div>
        `
      })
      new Vue({
        el: '#template',
        data: {
          name: 'donghai'
        },
        components: {
          'se-com': {
            props: ['param'],
            template: '#optioncompTemp'
          }
        },
        // 字符串模板，替换全部的模板，内联字符串模板
        template: `
        <ol>
          <tr is="my-component" :param="name"></tr>
          <tr is="se-com" :param="name" ></tr>
          <se-com :param="name"></se-com>
        </ol>
        `
      })
    </script>
  </body>
{% endcodeblock %}  

　　除了DOM模板和字符串模板之外，在vue中还存在着内联字符串模板以及{% raw %}JS脚本模板{% endraw %}模板，内联字符串模板指的是，在一个组件内部引用其它组件的时候，这个被引入的子组件也是直接用字符串表示，而非来自其它形式(如导入一个模板等)方式的引入，

{% codeblock lang:js %}

  new Vue({
        el: '#template',
        data: {
          name: 'donghai'
        },
        components: {
          'se-com': {
            props: ['param'],
            template: `<div>我是第二个组件{% raw %}{{param}}{% endraw %}</div>`
          }
        },
        // 字符串模板，替换挂载元素
        template: `
        <ol>
          <tr is="my-component" :param="name"></tr>
          <tr is="se-com" :param="name" ></tr>
          <se-com :param="name"></se-com>
        </ol>
        `
      })

{% endcodeblock %}  

　　JS脚本模板指的是使用script标签去声明一个组件的模板，

{% codeblock lang:html %}
  <body>
    <div id="app">
      <select>
        <option is="optioncomp"></option>
      </select>
    </div>
      <!--模板内容存放区域-->
    <script type="text/x-template" id="optioncompTemp">
      <option>a</option>
    </script>
    <script>
      new Vue({
        el: '#app',
        components: {
          'optioncomp': {
            template: '#optioncompTemp'
          }
        }
      })
    </script>
  </body>
{% endcodeblock %}  

　　但是如果我们使用文件模板，即xxx.vue的方式，在前端工程中就不存在上面提到的大小写的问题了。除此之外，vue模板中定义的组件在一些特定的标签下受到限制，例如***ul、ol、table、select***这样的元素里允许包含的元素有限制，而另一些像***option***这样的元素只能出现在某些特定元素的内部。下面这样的方式是不被允许的，

{% codeblock lang:html %}
  <table>
    <my-row>...</my-row>
  </table>
{% endcodeblock %}  

　　这时候自定义组件my-row会被当成无效的内容，这时候需要使用到特殊的is属性，

{% codeblock lang:html %}
  <table>
    <tr is="my-row"></tr>
  </table>
{% endcodeblock %}  

　　另外这个特殊的标签限制只会的一些内容有效，对于下面的情况，则可以在这些被限制的标签中使用自定义模板，

- 内联字符串模板
- 单文件组件 (.vue)
- JS脚本模板

### 静态传递和动态传递

　　我们知道可以这样传递到组件中一个静态的值，而且它总是一个字符串类型，

{% codeblock lang:html %}
  <blog-post title="My journey with Vue"></blog-post>
{% endcodeblock %}  

　　通过v-bind进行动态赋值，

{% codeblock lang:html %}

  <!-- 动态赋予一个变量的值 -->
  <blog-post v-bind:title="post.title"></blog-post>

  <!-- 动态赋予一个复杂表达式的值 -->
  <blog-post
    v-bind:title="post.title + ' by ' + post.author.name">
  </blog-post>

{% endcodeblock %}  

　　对于静态赋值，有时候我们想要传递一个数字类型或者布尔类型的时候，是无法做到的，所以这时候我们必须使用动态赋值的方式传递一个js表达式，

{% codeblock lang:html %}

  <!-- 即便 `42` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
  <!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
  <blog-post v-bind:likes="42"></blog-post>

  <!-- 包含该 prop 没有值的情况在内，都意味着 `true`。-->
  <blog-post is-published></blog-post>

  <!-- 即便 `false` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
  <!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
  <blog-post v-bind:is-published="false"></blog-post>

  <!-- 即便数组是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
  <!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
  <blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

  <!-- 即便对象是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
  <!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
  <blog-post
    v-bind:author="{
      name: 'Veronica',
      company: 'Veridian Dynamics'
    }"></blog-post>

{% endcodeblock %}  

　　如果你想要在组件中同时定义多个属性，但又不希望把它们都放置在一个对象中，这时候你可能需要写很多赋值语句，vue提供了不带参数的v-bind指令帮助你一次传递一个对象的所有属性，

{% codeblock lang:html %}

  <blog-post v-bind="post"></blog-post>

  等价于:

  <blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"></blog-post>

{% endcodeblock %}  

### 单向数据流

　　Vue中所有的数据在父组件和自组件中是单向下行绑定的，这意味着，父组件中修改传递到子组件中的prop时，子组件中的数据同样会被修改，但是反之却不行。这样的目的是为了防止子组件的改动影响到父组件中的数据，会导致应用的数据流向难以理解。Vue这样设计是很合理的，因为有时候我们不希望子组件的改动影响到父组件的数据，只有在必要的时候才去这么做，这样使我们对数据具备更多的可控性。

　　另外在Vue中如果你在子组件中修改了prop，那么控制台会提示一个警告，意味着你不应该这么做。从父组件传递到子组件的prop一般有下面两个用处，

　　1. 用于传递一个初始值，子组件依赖这个初始值去进行组件的渲染，并将其当作一个本地数据使用。这时候你应该拷贝一份数据到本地，
{% codeblock lang:js %}
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
{% endcodeblock %}  

　　2. 这个prop不需要修改，只是用于读取，但是子组件需要修改传递过来的数据源，这时候可以将其设置为一个计算属性，

{% codeblock lang:js %}
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
{% endcodeblock %}  

　　***对于对象类型或者数组类型的prop，在从父组件传递到组件中的时候，变量的类型是引用，也就是指向对象和数组的地址，这时候上面的单向数据流就不成立了。就是说单向数据流法则仅当你传递的是一个非对象和数组类型的值的时候才成立，否则子组件和父组件一样会将数据的修改影响到对方。***

### Prop验证

　　有时候我们在编写一个组件的时候，自己一般知道需要往里面传递什么类型的值，或者这个Prop的一些限制，但是后面的开发者并不知道这时候需要怎么传递，你可以为这些Prop添加一个验证帮助其它开发者检验错误，当prop验证失败的时候，(开发环境构建版本的)Vue 将会产生一个控制台的警告。如果需要对Prop检查错误，就必须使用对象的语法形式而非数组，

{% codeblock lang:js %}

  Vue.component('my-component', {
    props: {
      // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
      propA: Number,
      // 多个可能的类型
      propB: [String, Number],
      // 必填的字符串
      propC: {
        type: String,
        required: true
      },
      // 带有默认值的数字
      propD: {
        type: Number,
        default: 100
      },
      // 带有默认值的对象
      propE: {
        type: Object,
        // 对象或数组默认值必须从一个工厂函数获取
        default: function () {
          return { message: 'hello' }
        }
      },
      // 自定义验证函数
      propF: {
        validator: function (value) {
          // 这个值必须匹配下列字符串中的一个
          return ['success', 'warning', 'danger'].indexOf(value) !== -1
        }
      }
    }
  })

{% endcodeblock %}  

　　***Prop验证发生在组件创建前，也就是说，组件的Prop中的default和validator不可以使用定义在组件data、computed、methods中的属性和方法***

　　type可以是下面原生构造函数中的一个：

- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol

　　除了这些默认的构造函数，你也可以使用自定义构造函数进行类型的检查，

{% codeblock lang:js %}

function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

Vue.component('blog-post', {
  props: {
    author: Person
  }
})

{% endcodeblock %}  

### 非Prop特性

　　非Prop特性这个名字听起来不容易被理解，简单来说，就是指那些在组件中的props属性中没有被声明的，但是又在父组件中向子组件传递的属性。比如下面的例子：

{% codeblock lang:html %}
  <div id="app">
      <my-comp data-title="learn vue" class="mycls" style="color:red;"></my-comp>
  </div>
  <script>
      Vue.component('my-comp', {
          template: '<div>我是组件</div>'
      });
      new Vue({
          el: '#app'
      });
  </script>
{% endcodeblock %}  

　　这里的my-comp组件中并未定义data-title这个prop，但是又向my-comp标签传递了这个属性，这时候会在这个组件的根元素上添加这个属性，所以这个地方会最终被渲染为，

{% codeblock lang:html %}

  <div id="app">
    <div data-title="learn vue" class="mycls" style="color:red;"></div>
  </div>

{% endcodeblock %}

　　这里包括class和style都属于非Prop特性，但不同的是，对于class和style这类非Prop特性，vue有做特殊的处理，前面在Class和Style绑定的时候也提到了，模板中定义的class和style和在模板中传递的值会被合并而不是简单的覆盖。对于其它非Prop特性来说，如果在组件中定义了这个属性，又接着传递了该属性，那么这个非Prop属性会被传递的值覆盖，

{% codeblock lang:html %}
  <script>
      Vue.component('my-comp', {
          template: '<div type="inital">我是组件</div>'
      });
      new Vue({
          el: '#app'
      });
  </script>

  <div id="app">
      <my-comp type="changed"></my-comp>
  </div>

{% endcodeblock %}

　　最后这里会被渲染为，

{% codeblock lang:html %}

  <div id="app">
      <div type="changed"></div>
  </div>

{% endcodeblock %}

　　当然，有时候你不想要传递的非Prop属性覆盖掉组件中定义的值，你可以组件的选项中设置 inheritAttrs: false去达到这个目的，这样，所有的非Prop属性都不会出现在最后组件根元素的DOM节点上，覆盖也就根本不存在了，***(inheritAttrs: false不会影响class和style的绑定)***

{% codeblock lang:js %}

  Vue.component('my-component', {
    inheritAttrs: false,
    // ...
  })

{% endcodeblock %}

　　那么，我们如何去获取到这些非Prop属性的值呢，vue提供了[$attrs](https://vuejs.org/v2/api/#vm-attrs)为我们做到了这一点，$attrs是一个包含所有非Prop属性的对象，***(不包括class和style)***，如果这样如果我们想要将一个非Prop属性绑定到组件的非根元素上时，使用这个属性将变的非常方便，

{% codeblock lang:html %}

  Vue.component('base-input', {
    inheritAttrs: false,
    props: ['label', 'value'],
    template: `
      <label>
        {% raw %}{{ label }}{% endraw %}
        <input
          v-bind="$attrs"
          v-bind:value="value">
      </label>
    `
  })

  在base-input标签中

  <base-input
    v-model="username"
    required
    placeholder="Enter your username">
    </base-input>

{% endcodeblock %}

　　这里将会被渲染为，

{% codeblock lang:html %}

  <label>
      {
        "required": "",
        "placeholder": "Enter your username"
      }<input required="required" placeholder="Enter your username">
  </label>

{% endcodeblock %}

　　这样你使用基础自定义组件就像是原始的HTML元素一样，避免了不必要的代码逻辑***(添加多余的prop)***。

## 自定义事件

### 事件名

　　自定义事件名称不像Prop一样存在大小写转换的可能，我们需要精确的匹配一个自定义事件的名称，才可以触发自定义事件的监听器。比如手动触发一个事件，

{% codeblock lang:js %}

  this.$emit('myEvent')

{% endcodeblock %}

　　如果去监听这个自定义事件的kebab-case是不会有任何效果的，

{% codeblock lang:html %}

  <!-- 没有效果 -->
  <my-component v-on:my-event="doSomething"></my-component>

{% endcodeblock %}

　　除此之外，***在DOM模板中***，v-on指令后面如果定义的是myEvent的话，同样这里的myEvent也会被解析为myevent，这种情况下也会导致myEvent监听器无法被触发，所以不推荐使用camelCase的事件命名，尽可能使用kebab-case的命名方式。

### 自定义组件的v-model

　　我们知道，v-model指令可以将组件的行为数据同步到绑定到的data，而v-model的实现原理就是在我们需要在对应的组件上监听原生的DOM事件并使用$emit发出一个自定义事件，然后v-model会在这个对应的自定义事件的监听器中修改绑定的data。v-model默认监听的是原生的input事件以及原生DOM的value属性，但不同的输入组件的事件和属性值会有不同，如果需要改变它的默认行为可以这样做，

{% codeblock lang:html %}

  Vue.component('base-checkbox', {
    model: {
      prop: 'checked',
      event: 'change'
    },
    props: {
      checked: Boolean
    },
    template: `
      <input
        type="checkbox"
        v-bind:checked="checked"
        v-on:change="$emit('change', $event.target.checked)">`
  })

{% endcodeblock %}

　　注意，根据之前所述，如果你需要对checked使用v-bind，必须在props中进行声明。

### 绑定原生事件

　　我们都知道v-on可以绑定事件监听器，但是这个的指令的例子中很多在组件标签中监听的是emit的input、change事件啦，在组件的一些原生的html元素中监听的也是input、change事件，这时候开发者会感到迷惑，那么什么时候监听原生的事件什么时候监听的是自定义的事件呢？它们的用法好像看上去没有区别。这时候我们需要借助官方文档的力量帮我们解除迷惑，

> ***用在普通元素上时，只能监听原生 DOM 事件。用在自定义元素组件上时，也可以监听子组件触发的自定义事件。***

　　看到这里恍然大悟，这就是说，在普通元素上v-on只可以监听原生的事件，如果使用在自定义元素上的时候，两者都可以，这时候默认监听自定义事件，但是如果需要去监听原生的事件需要加上***.native***修饰符。

　　在监听原生事件的时候，监听器处理方法只有事件原生对象为唯一的参数。如果使用内联语句，在语句中可以访问一个$event属性，

{% codeblock lang:js %}

  v-on:click="handle('ok', $event)"

{% endcodeblock %}

　　在使用v-on监听自定义事件的时候，不像监听原生事件一样有一个事件原生对象，这种情况只存在一个从<span>$emit</span>传递过来的额外的参数，有趣的是，这两种方式都使用$event作为传递的变量名称，***(虽然都被写为$event，但是含义却大不一样，一个是原生的事件对象，一个是负载信息)***

{% codeblock lang:html %}

  <!-- 内联语句，$event只是负载信息，如果需要标识DOM，可以添加data-属性 -->
  <my-component @my-event="handleThis(123, $event)"></my-component>

  <!-- 也可以直接绑定到一个方法变量，第一个参数就是这个负载信息 -->
  this.$emit('give-advice',  { detail: detailInfo })

  <div id="emit-example-argument">
    <magic-eight-ball v-on:give-advice="showAdvice"></magic-eight-ball>
  </div>

  <!-- 这里的advice的值为  { detail: detailInfo } -->
  new Vue({
    el: '#emit-example-argument',
    methods: {
      showAdvice: function (advice) {
        alert(advice)
      }
    }
  })

{% endcodeblock %}

　　理解了v-on的使用方式，我们可以轻松的绑定一个原生事件，但是我们在自定义组件上使用v-on.native的时候，只会将这个事件绑定到组件的根元素上，对于有些情况来说，这样的绑定会失效，比如下面这样的自定义组件，

{% codeblock lang:html %}

  <label>
    {% raw %}{{ label }}{% endraw %}
    <input
      v-bind="$attrs"
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)">
  </label>

{% endcodeblock %}

　　姑且将其称之为base-input，如果我们在这个base-input上监听一个focus事件，

{% codeblock lang:html %}

  <base-input v-on:focus.native="onFocus"></base-input>

{% endcodeblock %}

　　对于这种情况，由于label并不是focusable元素，所以对这个标签添加focus监听器是没有作用的，这时候这个focus监听器并不会被添加到input元素上去，我们的v-on:focus.native也就会起不到任何作用。针对这种情况，Vue提供了一个 <span>$listeners</span>属性，$listeners是一个对象，包含了绑定到这个组件上的根元素的所有事件监听器，***(不包括通过.native修饰符添加的监听器，且$listeners仅在2.4+中可用)***，一个<base-input>如果有如下的定义，

{% codeblock lang:html %}

  <base-input v-on:mouseover="handleMouseOver" v-on:click="handleClick"></base-input>

{% endcodeblock %}

　　这样这个$listeners在组件内的值为，

{% codeblock lang:html %}

  {
    mouseover: handleMouseOver(event) { ... }
    click: handleClick(value) { ... },
  }

{% endcodeblock %}

　　所以如果我们需要在组件的子元素绑定一些原生事件，配合上计算属性可以进行一个自定义的添加，

{% codeblock lang:js %}

  Vue.component('base-input', {
    inheritAttrs: false,
    props: ['label', 'value'],
    computed: {
      inputListeners: function () {
        var vm = this
        // `Object.assign` 将所有的对象合并为一个新对象
        return Object.assign({},
          // 我们从父级添加所有的监听器
          this.$listeners,
          // 然后我们添加自定义监听器，
          // 或覆写一些监听器的行为
          {
            // 这里确保组件配合 `v-model` 的工作
            input: function (event) {
              vm.$emit('input', event.target.value)
            }
          }
        )
      }
    },
    template: `
      <label>
        {% raw %}{{ label }}{% endraw %}
        <input
          v-bind="$attrs"
          v-bind:value="value"
          v-on="inputListeners">
      </label>
    `
  })

{% endcodeblock %}

　　这样的话，我们在自定义组件上直接添加监听器就好像在组件子元素上直接添加了原生的监听器，看上去它们就像是一个元素。

### .sync 修饰符

　　之前我们已经在Prop中编写了Vue中组件的单向数据流向，但有时候我们希望改变这个特性，将数据进行双向绑定，使用v-model是一种方式，但其本质是通过自定义事件的监听器去实现的。v-model是针对组件的行为进行的双向绑定，对于一些更加通用的做法，Vue推荐使用update:myPropName的模式去达到这个目的。什么是update:myPropName模式呢？简而言之就是子组件内发出一个update:myPropName的事件，附带上myProp的新值，并在组件根元素上添加这个自定义事件的监听器进行修改。

{% codeblock lang:html %}

  <!-- 子组件中 -->
  this.$emit('update:title', newTitle)

  <!-- 根组件 -->
  <text-document
    v-bind:title="doc.title"
    v-on:update:title="doc.title = $event"
  ></text-document>

{% endcodeblock %}

　　为了方便，Vue提供了.sync 修饰符，也就是说，被.sync修饰符修饰过的变量，只需要在子组件中发出"update:title"这个事件就可以实现数据的双向绑定了。.sync的用法，

{% codeblock lang:html %}

  <text-document v-bind:title.sync="doc.title"></text-document>

  <!-- 设置多个prop的时候 -->
  <text-document v-bind.sync="doc"></text-document>

{% endcodeblock %}

　　完整的使用方式，

{% codeblock lang:html %}

  <div id="app">
    <p>{% raw %}{{ message }}{% endraw %}</p>
    <child :open.sync="message"></child>
  </div>

  <template id="child">
    <div>
      <input type="text" :value="open" @input="$emit('update:open', $event.target.value)">
      open: {% raw %}{{ open }}{% endraw %}
    </div>
  </template>

{% endcodeblock %}

　　说白了，.sync修饰符就是语法糖，

{% codeblock lang:js %}

  :open.sync="state"

  <!-- 相当于 -->

  :open="state" @update:open="state = $event"

{% endcodeblock %}

　　***注意带有 .sync 修饰符的 v-bind 不能和表达式一起使用，以及将 v-bind.sync 用在一个字面量的对象上，例如 v-bind.sync=”{ title: doc.title }”，是无法正常工作的，这是因为在vue需要考虑很多边缘情况***


## 插槽

　　***在vue 2.6+中，为插槽这部分内容引入v-slot指令去替代slot和slot-scope，slot和slot-scope已经被废弃，但是在vue2.x中仍然被支持。关于为什么弃用slot-scope，官方在[这里](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md)解释了***

### 插槽内容

　　在自定义组件中，虽然我们可以通过数据驱动去定义不同的组件，但是很多时候这种方式只能用于有限种情况的使用，或者说已知情况的定义。所以如果我们想要直观的在组件中内嵌一些元素，就像普通的HTML元素一样，并需要灵活的添加任意的元素类型和数目，这时候插槽的作用就体现出来了。Vue采用<slot>标签作为承载这样的内嵌元素的出口，比如你定义一个下面这样的组件，我们称其为navigation-link，

{% codeblock lang:html %}

  <a
    v-bind:href="url"
    class="nav-link">
    <slot></slot>
  </a>

{% endcodeblock %}

　　在使用组件的时候，

{% codeblock lang:html %}

  <navigation-link url="/profile">
    Your Profile
  </navigation-link>

{% endcodeblock %}

　　这样，在渲染的时候，<slot>部分就会被替代为"Your Profile"。在插槽内不仅可以添加字符串，还可以添加任意html原生元素和自定义组件标签，

{% codeblock lang:html %}

  <navigation-link url="/profile">
    <!-- 添加一个 Font Awesome 图标 -->
    <span class="fa fa-user"></span>
    Your Profile
  </navigation-link>

  <!-- 甚至包括自定义组件 -->

  <navigation-link url="/profile">
    <!-- 添加一个图标的组件 -->
    <font-awesome-icon name="user"></font-awesome-icon>
    Your Profile
</navigation-link>

{% endcodeblock %}

　　如果在navigation-link组件的定义中没有包含slot标签，那么该组件标签起始和结束之前的任何内容都会被丢弃。

### 后备内容

　　对于一个插槽来说，后备内容是在组件被使用时使用者并未为插槽提供任何内容时显示的内容，可以被称为插槽默认内容。比如定义一个submit-buton组件，

{% codeblock lang:html %}

  <button type="submit">
    <slot>Submit</slot>
  </button>

{% endcodeblock %}

　　默认情况下如果我们在引用这个组件时，直接像下面这样使用，

{% codeblock lang:html %}

  <submit-button></submit-button>

{% endcodeblock %}

　　会被渲染为，

{% codeblock lang:html %}

  <button type="submit">
    Submit
  </button>

{% endcodeblock %}

　　当提供内容的时候，

{% codeblock lang:html %}

  <submit-button>
    Save
  </submit-button>

{% endcodeblock %}

　　则这个提供的内容将会被渲染从而取代后备内容：

{% codeblock lang:html %}

  <button type="submit">
    Save
  </button>

{% endcodeblock %}


###　具名插槽

　　有时候我们需要将多个插槽定义在组件的不同位置，比如下面的base-layout组件，

{% codeblock lang:html %}

  <div class="container">
    <header>
      <!-- 我们希望把页头放这里 -->
    </header>
    <main>
      <!-- 我们希望把主要内容放这里 -->
    </main>
    <footer>
      <!-- 我们希望把页脚放这里 -->
    </footer>
  </div>

{% endcodeblock %}

　　对于这种情况，我们在定义slot的时候，需要用到一个特殊的属性name，

{% codeblock lang:html %}

  <div class="container">
    <header>
       <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>

{% endcodeblock %}

　　不带name的slot出口会带有隐含的名字"default"，对于使用了具名插槽的组件，需要配合template元素使用v-slot指令提供插槽名称的方式去提供插槽内容，

{% codeblock lang:html %}

  <base-layout>
    <template v-slot:header>
      <h1>Here might be a page title</h1>
    </template>

    <p>A paragraph for the main content.</p>
    <p>And another one.</p>

    <template v-slot:footer>
      <p>Here's some contact info</p>
    </template>
  </base-layout>

  <!-- 下面和上面的方式结果相同，只不过是一个显示的指定了default，另一个使用默认的方式提供默认插槽的内容 -->

  <base-layout>
    <template v-slot:header>
      <h1>Here might be a page title</h1>
    </template>

    <template v-slot:default>
      <p>A paragraph for the main content.</p>
      <p>And another one.</p>
    </template>

    <template v-slot:footer>
      <p>Here's some contact info</p>
    </template>
  </base-layout>

{% endcodeblock %}

　　注意 v-slot 只能添加在一个template标签上 (只有一种例外情况，下文会提到，为默认插槽添加插槽Prop的时候，在组件标签上添加v-slot)，这一点和已经废弃的 slot 特性不同。

### 编译作用域

　　有时候你可能会想在插槽中使用一些这个插槽所属组件内部的数据，比如，

{% codeblock lang:html %}

  <navigation-link url="/profile">
    Clicking here will send you to: {% raw %}{{ url }}{% endraw %}
    <!--
    这里的 `url` 会是 undefined，因为 "/profile" 是
    _传递给_ <navigation-link> 的而不是
    在 <navigation-link> 组件*内部*定义的。
    -->
  </navigation-link>

{% endcodeblock %}

　　官方对这个编译作用域的解释有点难以理解，而且也会涉及到之前的后备插槽的内容，所以我有意将后备插槽以及具名插槽提到这部分前面，便于结合这两部分一起解释这个编译作用域。

　　先说结论，我们使用插槽添加内容时，这部分我们自定义的内容只可以访问和它本身所处同一模板文件的数据作用域，而不可以访问这个插槽所作用的组件内部的数据作用域。而当我们使用后备内容的时候，后备内容是处于在组件内部的模板定义，所以后备内容只可以访问组件内部的数据作用域，不可以访问组件外部的数据作用域，这两者恰好相反。这时候就很好的可以理解官方的一句话了，

> 父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

　　我们通过下面的例子可以去理解这种情况，在组件child定义四个具名插槽，前两个个具名插槽都的后备内容分别使用内部和外部两个变量，后面两个的插槽都在父级传递访问内部和外部两个值，

{% codeblock lang:js %}
<!-- 定义下面的内容 -->
  Vue.component('child', {
  	template: `
    	<div>
        <slot>{% raw %}{{ innerMsg }}{% endraw %}</slot>
        <slot name="second">{% raw %}{{ outerMsg }}{% endraw %}</slot>
        <slot name="third"></slot>
        <slot name="forth"></slot>
      </div>`,
    data(){
    	return {
     		innerMsg: "内部定义的第一个值",
        secondInnerMsg: "内部定义的第二个值"
      }
    }
  });

  new Vue({
    el: '#app',
    data: {
      outerMsg: '外部定义的一个值',
      secondOuterMsg: "外部定义的第二个值"
    }
  })

{% endcodeblock %}

　　以及对应的html模板，

{% codeblock lang:html %}

  <script src="https://unpkg.com/vue"></script>

  <div id="app">
    <child>
      <template v-slot:default></template>
      <template v-slot:second></template>
      <template v-slot:third>{% raw %}{{ secondInnerMsg }}{% endraw %}</template>
      <template v-slot:forth>{% raw %}{{ secondOuterMsg }}{% endraw %}</template>
    </child>
  </div>

{% endcodeblock %}

　　渲染结果，

{% codeblock lang:html %}

  <div id="app">
    <div>内部定义的第一个值   外部定义的第二个值</div>
  </div>

  <!-- 另外控制有如下的显示，
    Property or method "outerMsg" is not defined on the instance but referenced during render
    Property or method "secondInnerMsg" is not defined on the instance but referenced during render.
   -->

{% endcodeblock %}


### 作用域插槽

　　从上面的解释我们知道，后备插槽和自定义的插槽具有独立的作用域，互相都无法访问彼此的作用内容。但数据是可以从父组件流向自组件的，也就是说，虽然子组件中不存在对应的数据，但是可以当数据从父组件中传递到子组件后，在后备插槽中也可以获取到了。但是问题是父组件中无法获取到子组件的数据，如果当我们需要在自定义插槽中使用到子组件的数据的时候，这时候需要借助插槽Prop，也就是作用域插槽，借助官方的例子来说明作用域插槽的使用，有一个current-user组件，它的插槽中存在这样一个后备插槽，后备内容显示用户的名，

{% codeblock lang:html %}

  <span>
    <slot>{% raw %}{{ user.lastName }}{% endraw %}</slot>
  </span>

{% endcodeblock %}

　　如果正常情况下我们想要它显示用户的姓，但是user是组件内的值，下面的做法肯定是没有作用的，

{% codeblock lang:js %}

  {% raw %}{{ user.firstName }}{% endraw %}

{% endcodeblock %}

　　为了让user在父级的插槽内容中可用，我们可以将user作为<slot>元素的一个特性绑定上去：

{% codeblock lang:html %}

  <span>
    <slot v-bind:user="user">
      {% raw %}{{ user.lastName }}{% endraw %}
    </slot>
  </span>

{% endcodeblock %}

　　在组件内部的插槽处绑定了属性之后，可以给v-slot带上一个值来定义我们提供的插槽prop的名字，插槽prop对象的命名可以随意，没有过多的约束，这里我们将其命名为slotProps，之后可以使用这个插槽prop对象去访问之前绑定在插槽上的prop，

{% codeblock lang:html %}

  <current-user>
    <template v-slot:default="slotProps">
      {% raw %}{{ slotProps.user.firstName }}{% endraw %}
    </template>
  </current-user>

{% endcodeblock %}

　　如果被提供的内容只有默认插槽的时候，可以直接将插槽prop对象的命名放到组件的标签上，

{% codeblock lang:html %}

  <current-user v-slot:default="slotProps">
    {% raw %}{{ slotProps.user.firstName }}{% endraw %}
  </current-user>

<!-- 由于是默认模板，还可以简写为 -->

  <current-user v-slot="slotProps">
    {% raw %}{{ slotProps.user.firstName }}{% endraw %}
  </current-user>
　　
{% endcodeblock %}

　　上面简写的方式仅仅可用于仅存在默认插槽的情况，不可以和具名插槽混用，因为每个插槽都有自己独立的Prop对象，这样会导致插槽Prop作用域不明确，如果存在多个插槽的情况，应该使用完整的基于template标签的语法：

{% codeblock lang:html %}

  <current-user>
    <template v-slot:default="slotProps">
      {% raw %}{{ slotProps.user.firstName }}{% endraw %}
    </template>

    <template v-slot:other="otherSlotProps">
      ...
    </template>
  </current-user>

{% endcodeblock %}

　　作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里，在环境支持的情况下(单文件组件或现代浏览器)，另一种获取插槽prop对象的方式是对其进行解构赋值，

{% codeblock lang:html %}

  <current-user v-slot="{ user }">
    {% raw %}{{ user.firstName }}{% endraw %}
  </current-user>

  <!-- 使用es6结构赋值对prop进行重命名 -->
  <current-user v-slot="{ user: person }">
    {% raw %}{{ person.firstName }}{% endraw %}
  </current-user>

  <!-- 在组件内部值不存在的时候，使用结构赋值传递一个默认值 -->
  <current-user v-slot="{ user = { firstName: 'Guest' } }">
    {% raw %}{{ user.firstName }}{% endraw %}
  </current-user>

{% endcodeblock %}

### 动态插槽名

　　在2.6.0+中，可以在v-slot上使用动态指令参数，定义动态的插槽名，

{% codeblock lang:html %}

  <base-layout>
    <template v-slot:[dynamicSlotName]>
      ...
    </template>
  </base-layout>

{% endcodeblock %}

### 具名插槽的缩写

　　和v-on以及v-bind一样，v-slot也有缩写，即把参数之前的所有内容(v-slot:)替换为字符#。例如v-slot:head，可以被重写为#header，

{% codeblock lang:html %}

  <base-layout>
    <template #header>
      <h1>Here might be a page title</h1>
    </template>

    <p>A paragraph for the main content.</p>
    <p>And another one.</p>

    <template #footer>
      <p>Here's some contact info</p>
    </template>
  </base-layout>

{% endcodeblock %}

　　该缩写只其在有参数的时候才可用，

{% codeblock lang:html %}

  <!-- 这样会触发一个警告，且这样的语法是无效的 -->
  <current-user #="{ user }">
    {% raw %}{{ user.firstName }}{% endraw %}
  </current-user>

  <!-- 正确的写法为: -->
  <current-user #default="{ user }">
    {% raw %}{{ user.firstName }}{% endraw %}
  </current-user>

{% endcodeblock %}

### 废弃的语法

　　前文提过，v-slot是在vue2.6+被支持的语法，之前的语法可以去参照[官方文档](https://cn.vuejs.org/v2/guide/components-slots.html#%E5%BA%9F%E5%BC%83%E4%BA%86%E7%9A%84%E8%AF%AD%E6%B3%95)。


## 动态组件 & 异步组件

　　有时候，我们可能在一个元素上根据条件显示不同的组件，这时候我们可能会想要使用v-if去根据数据的不同动态渲染不同的组件。除此之外，同时vue还提供了动态组件去实现这个需求，一个经典的案例如下，

{% codeblock lang:html %}

  <!-- 已经注册的组件 -->
  Vue.component('tab-home', {
  	template: '<div>Home component</div>'
  })

  Vue.component('tab-posts', {
  	template: '<div>Posts component</div>'
  })

  Vue.component('tab-archive', {
  	template: '<div>Archive component</div>'
  })

  <div id="dynamic-component-demo" class="demo">
    <button
      v-for="tab in tabs"
      v-bind:key="tab.name"
      v-bind:class="['tab-button', { active: currentTab.name === tab.name }]"
      v-on:click="currentTab = tab"
    >{% raw %}{{ tab.name }{% endraw %}}</button>

    <component
      v-bind:is="currentTab.component"
      class="tab"
    ></component>

  </div>

  <!-- vue根组件 -->
  new Vue({
    el: '#dynamic-component-demo',
    data: {
      currentTab: 'Home',
      tabs: ['Home', 'Posts', 'Archive']
    },
    computed: {
      currentTabComponent: function () {
        return 'tab-' + this.currentTab.toLowerCase()
      }
    }
  })

{% endcodeblock %}

　　通过在这个特殊的component标签上添加is属性，我们实现了在一个元素上动态渲染不同组件的功能。这里的currentTabComponent不仅是可以指向已经注册组件的名称，还可以指向一个组件的选项对象，另一种使用例子可以看[这里](https://jsfiddle.net/chrisvfritz/b2qj69o1/)。

　　从上面我们得知我们可以使用动态组件根据条件在同一个位置渲染不同的组件，但是在不同的组件间进行切换的时候会导致这个组件会被重新渲染，这样在之前的页面进行的一些修改将不会被保留下来。要保留之前组件的状态的，可以使用vue提供的keep-alive元素，

{% codeblock lang:html %}

  <!-- 失活的组件将会被缓存！-->
  <keep-alive>
    <component v-bind:is="currentTabComponent"></component>
  </keep-alive>

{% endcodeblock %}

　　使用例子可以看[这里](https://jsfiddle.net/chrisvfritz/Lp20op9o/)。

　　在大型应用中，我们可能需要将应用分割成一些小的块，并且只有在需要的时候才从服务器中加载，而不是一次将所有需要的内容都加载下来。而通过这种方式进行条件加载的组件被称为异步组件，vue通过允许你使用一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义，且vue只有在这个组件需要被渲染的时候才发触发这个工厂函数的执行，并将结果缓存起来提供到接下来的重渲染，

{% codeblock lang:js %}

  Vue.component('async-example', function (resolve, reject) {
    setTimeout(function () {
      // 向 `resolve` 回调传递组件定义
      resolve({
        template: '<div>I am async!</div>'
      })
    }, 1000)
  })

{% endcodeblock %}

　　但这种编写在vue的方式并不实用，因为我们依然没有减少下载js代码的体积，并让代码的维护性和重构性变的很不友好，所以一般需要配合[webpack的code-splitting功能](https://webpack.js.org/guides/code-splitting/)进行一起使用，

{% codeblock lang:js %}

  Vue.component('async-webpack-example', function (resolve) {
    // 这个特殊的 `require` 语法将会告诉 webpack
    // 自动将你的构建代码切割成多个包，这些包
    // 会通过 Ajax 请求加载
    require(['./my-async-component'], resolve)
  })

{% endcodeblock %}

　　在上述情况下，你也使用返回一个Promise的方式去异步加载这个组件，

{% codeblock lang:js %}

  <!-- 全局注册 -->
  Vue.component(
    'async-webpack-example',
    // 这个 `import` 函数会返回一个 `Promise` 对象。
    () => import('./my-async-component')
  )
  <!-- 局部注册 -->
  new Vue({
    // ...
    components: {
      'my-component': () => import('./my-async-component')
    }
  })

{% endcodeblock %}

　　在vue 2.3 +中，加载异步组件的时候，vue支持返回一个包含如下格式的对象去处理在加载过程中组件显示的内容，***(如果你希望在Vue-Router中使用下面的语法的话，需要vue-router 2.4 + )***

{% codeblock lang:js %}

  const AsyncComponent = () => ({
    // 需要加载的组件 (应该是一个 `Promise` 对象)
    component: import('./MyComponent.vue'),
    // 异步组件加载时使用的组件
    loading: LoadingComponent,
    // 加载失败时使用的组件
    error: ErrorComponent,
    // 展示加载时组件的延时时间。默认值是 200 (毫秒)
    delay: 200,
    // 如果提供了超时时间且组件加载也超时了，
    // 则使用加载失败时使用的组件。默认值是：`Infinity`
    timeout: 3000
  })

{% endcodeblock %}


## 处理边界情况

### 访问元素 & 组件

　　在大多数情况下，在一个Vue应用中是不需要直接操作DOM的，但是对于一些情况则不是这样，比如当我们引入一个三方组件的时候，这时候很有可能需要直接操作DOM去达到业务需求。

#### 访问根实例

　　在每个new Vue实例的组件中，其根实例可以通过$root属性进行访问。比如在下面的例子中，

{% codeblock lang:js %}

  // Vue 根实例
  new Vue({
    data: {
      foo: 1
    },
    computed: {
      bar: function () { /* ... */ }
    },
    methods: {
      baz: function () { /* ... */ }
    }
  });

  <!-- 通过$root，所有的子组件都可以将这个实例当作一个全局的store来使用 -->

  // 获取根组件的数据
  this.$root.foo

  // 写入根组件的数据
  this.$root.foo = 2

  // 访问根组件的计算属性
  this.$root.bar

  // 调用根组件的方法
  this.$root.baz()

  <!-- 对于 demo 或非常小型的有少量组件的应用来说这是很方便的。
      不过这个模式扩展到中大型应用来说就不然了。
      因此在绝大多数情况下，我们强烈推荐使用 Vuex 来管理应用的状态。
   -->

{% endcodeblock %}

#### 访问父级组件实例

　　和$root类似，$parent属性可以用来从一个子组件访问父组件的实例。它提供了一种机会，可以在后期随时触达父级组件，以替代将数据以 prop 的方式传入子组件的方式。***(将数据传入自组件的方式达到自组件调用父组件方法或修改属性的方式容易让应用变的难以被理解，并且prop的作用应该是父组件需要传递到子组件的初始数据，而不是用于父子组件间的互相通讯。其实即便是通过$root的方式，也会容易使应用很难被理解，但是这种方式相对来说是要优于使用Prop的方式。)***

{% codeblock lang:js %}

  <google-map>
    <google-map-marker v-bind:places="vueConfCities"></google-map-marker>
  </google-map>

  <!-- 在子组件调用父组件方法，并传递一个方法引用在父组件的上下文中被调用 -->

  Vue.component('google-map-marker', {
    props: ['places'],
    created: function () {
      var vm = this
      vm.$parent.getMap(function (map) {
        vm.places.forEach(function (place) {
          new google.maps.Marker({
            position: place.position,
            map: map
          })
        })
      })
    },
    render (h) {
      return null
    }
  })

{% endcodeblock %}

#### 访问子组件实例或子元素

　　尽管存在prop和事件，有的时候你仍可能需要在JavaScript里直接访问一个子组件。为了达到这个目的，你可以通过ref特性为这个子组件赋予一个ID引用。ref被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的$refs对象上。例如：

{% codeblock lang:html %}

  <base-input ref="usernameInput"></base-input>

{% endcodeblock %}

　　现在在你已经定义了这个ref的组件里，你可以使用访问子组件实例：

{% codeblock lang:js %}

  this.$refs.usernameInput

{% endcodeblock %}

　　同样在我们的这个base-input子组件中，也可以对组成它的基本元素加上ref属性。如果在普通的DOM元素上使用ref，那么这个引用指向的就是DOM元素。如果用在子组件上，引用就指向组件实例：

{% codeblock lang:html %}

  <!-- 在base-input的模板DOM元素上添加1 -->
  <input ref="input">

  <!-- `vm.$refs.p` 是一个DOM对象 -->
  <p ref="p">hello</p>

  <!-- `vm.$refs.child` 是一个组件实例 -->
  <child-component ref="child"></child-component>

{% endcodeblock %}

　　***$refs 只会在组件渲染完成之后生效，并且它们不是响应式的。关于ref 注册时间的重要说明：因为 ref 本身是作为渲染结果被创建的，在初始渲染的时候你不能访问它们 - 它们还不存在！$refs 也不是响应式的，因此你不应该试图用它在模板中做数据绑定。***


#### 依赖注入

　　前面我们已经提到了父组件和子组件互相持有引用的方式，这类场景在cs的应用程序中很常见，在cs程序的开发中，经常使用一些依赖注入框架解决这类问题，比如在Android中就有Dagger等框架。除此之外，在Web服务端也有此类的需求，在后端经常会出现业务之间的交叉，为了减少代码之间的耦合度，也会使用一些依赖注入框架，使用比较多的应该就是Spring IOC了。看来vue也是仿照此类模式的实现，我们来看看Vue中的依赖注入的使用方式，在之前的例子中，假设父组件和子组件之间又需要加入一个中间组件，

{% codeblock lang:html %}

  <google-map>
    <google-map-region v-bind:shape="cityBoundaries">
      <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
    </google-map-region>
  </google-map>

{% endcodeblock %}

　　由于出现了google-map-region这个组件，并且在这个组件里，所有 <google-map> 的后代都需要访问一个 getMap 方法，以便知道要跟哪个地图进行交互。不幸的是，使用 $parent 属性无法很好的扩展到更深层级的嵌套组件上。这也是依赖注入的用武之地，它用到了两个新的实例选项：provide 和 inject。

　　provide 选项允许我们指定我们想要提供给后代组件的数据/方法。在这个例子中，就是 <google-map> 内部的 getMap 方法：

{% codeblock lang:js %}

  provide: function () {
    return {
      getMap: this.getMap
    }
  }

{% endcodeblock %}

　　然后在任何后代组件里，我们都可以使用 inject 选项来接收指定的我们想要添加在这个实例上的属性：

{% codeblock lang:js %}

  inject: ['getMap']

{% endcodeblock %}

　　相比$parent来说，这个用法可以让我们在任意后代组件中访问getMap，而不需要在每个子组件间中大量的使用$parent。并且这种方式不需要担心我们可能会改变/移除一些子组件依赖的东西，在对原有逻辑进行很小的改动的情况下调用父组件的方式和属性。

　　***然而，依赖注入还是有负面影响的。它将你应用程序中的组件与它们当前的组织方式耦合起来，使重构变得更加困难。同时所提供的属性是非响应式的。这是出于设计的考虑，因为使用它们来创建一个中心化规模化的数据跟使用 $root做这件事都是不够好的。如果你想要共享的这个属性是你的应用特有的，而不是通用化的，或者如果你想在祖先组件中更新所提供的数据，那么这意味着你可能需要换用一个像Vuex这样真正的状态管理方案了。***

### 程序化的监听器

　　我们已经知道可以使用v-on监听$emit发出的事件，但是有时候我们想要在程序中动态的添加监听器，这时候就可以:

- 通过 $on(eventName, eventHandler) 侦听一个事件
- 通过 $once(eventName, eventHandler) 一次性侦听一个事件
- 通过 $off(eventName, eventHandler) 停止侦听一个事件

　　如果你需要用的一个三方组件，并在组件挂载的时候进行创建，组件销毁的时候同时对这个三方组件进行销毁，你很有可能编写下面的代码，

{% codeblock lang:js %}

  // 一次性将这个日期选择器附加到一个输入框上
  // 它会被挂载到 DOM 上。
  mounted: function () {
    // Pikaday 是一个第三方日期选择器的库
    this.picker = new Pikaday({
      field: this.$refs.input,
      format: 'YYYY-MM-DD'
    })
  },
  // 在组件被销毁之前，
  // 也销毁这个日期选择器。
  beforeDestroy: function () {
    this.picker.destroy()
  }

{% endcodeblock %}

　　但是这样的方式会使得组件实例持有三方组件的引用，但其实理论上组件实例是没有必要去持有这样的一个引用，这样的增加组件实例的属性的做法显得有些多余。第二个问题是我们的建立代码和清理代码分离，这样如果之后我们需要清理这个三方组件就需要在组件中清理所有销毁的相关内容。

　　为了解决上面的两个问题，你应该通过一个程序化的侦听器解决这两个问题：

{% codeblock lang:js %}

  mounted: function () {
    var picker = new Pikaday({
      field: this.$refs.input,
      format: 'YYYY-MM-DD'
    })

    this.$once('hook:beforeDestroy', function () {
      picker.destroy()
    })
  }

{% endcodeblock %}

　　这样，只需要在组件销毁的时候，$emit一个hook:beforeDestroy事件就可以清理这个三方组件了，甚至可以将它们包裹在一个方法里，即使重复引用多个三方组件，也可以一次清理干净，

{% codeblock lang:js %}

  mounted: function () {
    this.attachDatepicker('startDateInput')
    this.attachDatepicker('endDateInput')
  },
  methods: {
    attachDatepicker: function (refName) {
      var picker = new Pikaday({
        field: this.$refs[refName],
        format: 'YYYY-MM-DD'
      })

      this.$once('hook:beforeDestroy', function () {
        picker.destroy()
      })
    }
  }

{% endcodeblock %}

　　这里需要理解清楚一点，当我们对同一个事件绑定多个处理函数时，在低版本的vue中是不支持使用v-on的数组形式的，并且我们也不可以在组件的标签上定义重复的v-on属性，所以这时候可以使用$on或者$once这样的方式动态添加多个监听器，这样所有的监听器都可以得到添加和执行，***需要搞清楚的一个概念是一个事件可以有多个不同的监听器的，执行顺序是先添加先被执行***。

　　***注意 Vue 的事件系统不同于浏览器的 EventTarget API。尽管它们工作起来是相似的，但是 $emit、$on, 和 $off 并不是 dispatchEvent、addEventListener 和 removeEventListener 的别名。***


### 循环引用

#### 递归组件

　　组件是可以在它们自己的模板中调用自身的。不过它们只能通过 name 选项来做这件事：

{% codeblock lang:js %}

  name: 'unique-name-of-my-component'

  <!-- 当你使用 Vue.component 全局注册一个组件时，这个全局的 ID 会自动设置为该组件的 name 选项。 -->

  Vue.component('unique-name-of-my-component', {
    // ...
  })

{% endcodeblock %}

　　虽然递归组件看上去很实用，但是稍有不慎，就有可能导致一个无限循环，

{% codeblock lang:js %}

  name: 'stack-overflow',
  template: '<div><stack-overflow></stack-overflow></div>'

{% endcodeblock %}

　　类似上述的组件将会导致“max stack size exceeded”错误，所以请确保递归调用是条件性的 (例如使用一个最终会得到 false 的 v-if)。

#### 组件之间的循环引用

　　假设你需要构建一个文件目录树，像访达或资源管理器那样的。你可能有一个 <tree-folder> 组件，模板是这样的：

{% codeblock lang:html %}

  <p>
    <span>{% raw %}{{ folder.name }}{% endraw %}</span>
    <tree-folder-contents :children="folder.children"/>
  </p>

{% endcodeblock %}

　　<tree-folder-contents> 组件，

{% codeblock lang:html %}

  <ul>
    <li v-for="child in children">
      <tree-folder v-if="child.children" :folder="child"/>
      <span v-else>{% raw %}{{ child.name }}{% endraw %}</span>
    </li>
  </ul>

{% endcodeblock %}

　　当你仔细观察的时候，你会发现这些组件在渲染树中互为对方的后代和祖先——一个悖论！当通过 Vue.component 全局注册组件的时候，这个悖论会被自动解开。如果你是这样做的，那么你可以跳过这里。然而，如果你使用一个模块系统依赖/导入组件，例如通过 webpack 或 Browserify，你会遇到一个错误：

>  Failed to mount component: template or render function not defined.

　　为了解释这里发生了什么，我们先把两个组件称为A和B。模块系统发现它需要A，但是首先A依赖B，但是B又依赖A，但是A又依赖B，如此往复。这变成了一个循环，不知道如何不经过其中一个组件而完全解析出另一个组件。为了解决这个问题，我们需要给模块系统一个点，在那里“A 反正是需要B的，但是我们不需要先解析B。”


　　在我们的例子中，把 <tree-folder> 组件设为了那个点。我们知道那个产生悖论的子组件是 <tree-folder-contents> 组件，所以我们会等到生命周期钩子 beforeCreate 时去注册它：

{% codeblock lang:html %}

  beforeCreate: function () {
    this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
  }

  <!-- 或者，在本地组件注册时候，你可以使用 webpack 的异步 import： -->
  components: {
    TreeFolderContents: () => import('./tree-folder-contents.vue')
  }

  <!-- 这样问题就解决了！ -->

{% endcodeblock %}


### 控制更新

#### 强制更新

　　你可能还没有留意到数组或对象的变更检测注意事项，或者你可能依赖了一个未被Vue的响应式系统追踪的状态。

　　然而，如果你已经做到了上述的事项仍然发现在极少数的情况下需要手动强制更新，那么你可以通过 $forceUpdate 来做这件事。比如Vue就没法检测对Map的追踪，不过这个问题在3.0版本得到添加。

#### 通过 v-once 创建低开销的静态组件

　　渲染普通的 HTML 元素在 Vue 中是非常快速的，但有的时候你可能有一个组件，这个组件包含了大量静态内容。在这种情况下，你可以在根元素上添加 v-once 特性以确保这些内容只计算一次然后缓存起来，就像这样：***(虽然响应式很好用，但也要使用在适当的地方哦)***

{% codeblock lang:html %}

  Vue.component('terms-of-service', {
    template: `
      <div v-once>
        <h1>Terms of Service</h1>
        ... a lot of static content ...
      </div>
    `
  })

{% endcodeblock %}

　　***再说一次，试着不要过度使用这个模式。当你需要渲染大量静态内容时，极少数的情况下它会给你带来便利，除非你非常留意渲染变慢了，不然它完全是没有必要的——再加上它在后期会带来很多困惑。例如，设想另一个开发者并不熟悉 v-once 或漏看了它在模板中，他们可能会花很多个小时去找出模板为什么无法正确更新。***


## 小结

　　这篇在十月份就开始写了，但是一直到十一月差不多中旬才发出来，效率实在是底下，虽然其中有很多其它的事情造成了一些拖延，但是这个速度还是不能被接受。这段时间差不多将近一个半月，陆陆续续抽时间将vue-router、vuex、vue的文档刷了一遍，vue还有小部分没有看，算是对之前的查漏补缺吧，也知道了很多之前没有刻意去了解过的一些细节。接下来的计划大概是先把vue剩下的内容刷完，然后再去研究webpack的文档，webpack实在是太重要了，简直就是前端项目的基石。也不知道等自己刷完这两个内容要多久，不过看了下接下来的开发计划，自己应该是挺闲的。应该可以在年前将这些都搞定。

　　再接下来的计划可能就很明确了，刷文档的目的当然还是为了开发做铺垫，很多人可能会说我不刷文档也可以愉快的开发呀。但是个人觉得这种方式对于职业来说太不靠谱，很多东西都理解的不透彻，在需求当头的时候就看能用就用了，也不考虑这样做的一些后果，或者说弊端。这对工程来说就是不负责的表现，自己也看了太多这种例子，对这种行为个人是有一些鄙视成分在里面的，虽然人都有一个成长的过程，但我觉得这是习惯问题，或者说态度问题。看多了这种粗制滥造的代码之后只能说，不可能要求每个人做到完美，对自己要求严格就行，不让自己难受就好了。

　　不知道还会不会写技术博客，自己有点儿动摇，因为有道笔记对我来说明显比md形式的博客更加方便，写完这篇之后自己确实有些动摇了，因为这对自己来说就像是一些笔记，而笔记是只适合个人翻阅的。

　　em...罗马不是一天建成的，早点休息吧。
