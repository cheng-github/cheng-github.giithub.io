layout: "post"
title: "Vue基础-条件列表渲染"
date: "2019-04-21 14:22"
categories:
- [UI,VUE]
tags:
- [TECHNOLOGY]
thumbnail: http://swcheng.com/images/vuelogo.png
---
　　创建一个下面的vue对象:
<!-- more -->
{% codeblock lang:js %}
  Vue.component('my-component',{
    template: '<li>This is a todo</li>',
    props: ['item', 'index']
  });

  var app = new Vue({
    el: "#app",
    data: {
        awesome: true,
        notawesome: true,
        ok: true,
        loginType: "username",
        showornot: false,
        items: [
          { message: 'Foo' },
          { message: 'Bar' }
        ],
        parentMessgae: "parent",
        forObject: {
          firstName: "John",
          lastName: "Sonw",
          age: 40
        }
    },
    methods: {
      toggleLoginType: function(){
        if(this.loginType == "username")
          this.loginType = "email"
        else
          this.loginType = "username"
      }
    }
  });
{% endcodeblock %}

### 条件渲染
　　使用v-if指令进行条件渲染，当属性值为true的时候显示内容否则不显示。
{% codeblock lang:html %}
  <p v-if="awesome">I was rendered by if command</p>
{% endcodeblock %}
　　当然也可以使用else，甚至else-if,但是需要注意的是，这些语句必须书写在一起。
{% codeblock lang:html %}
  <p v-if="loginType == 'username'">Username login</p>
  <!-- v-else必须紧跟v-if 或 v-else-if后面，否则不会生效 -->
  <p v-else-if="loginType == 'email'">Email login</p>
  <p v-else>No, I am the king</p>
{% endcodeblock %}
　　如果想要一次渲染多个元素，可以使用template标签:
{% codeblock lang:html %}
  <template v-if="ok">
    <h1>Title</h1>
    <p>Paragraph 1</p>
    <p>Paragraph 2</p>
  </template>
  <template v-else>
    <h1>Another</h1>
    <p>Paragraph 21</p>
    <p>Paragraph 22</p>
  </template>
{% endcodeblock %}
　　渲染结果:
{% codeblock lang:html %}
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
{% endcodeblock %}
　　Vue为了尽可能高效的渲染元素，通常会复用现有元素而不是从头开始渲染，如果你需要vue重头开始渲染元素，那么你需要为它们指定key属性。
{% codeblock lang:html %}
  <template v-if="loginType === 'username'">
    <label>Username</label>
    <input placeholder="Enter your username" key="username-input">
  </template>
  <template v-else>
    <label>Email</label>
    <input placeholder="Enter your email address" key="email-input">
  </template>
{% endcodeblock %}

### 使用v-show
　　如果我们需要经常切换某个元素是否可见，为了效率起见我们应该使用v-show指令而非v-for。因为v-for是惰性的，只有当条件成立的时候才会渲染元素，而v-show则是无论条件是否成立都会一开始就渲染元素。并且v-show每次切换仅仅修改css的display属性，而v-if则会使条件块内的事件监听器和子组件适当地被销毁和重建，这样明显每次切换开销都要大于v-show。v-show的简单使用用例如下。
{% codeblock lang:html %}
  <!-- 对于一些经常需要改变的元素可以使用v-show，即只改变display的值 -->
  <h1 v-show="showornot">我是vshow代表</h1>
{% endcodeblock %}

### 列表渲染
　　使用v-for指令达到渲染一个列表的目的，并且需要使用item in items形式的语法，你也可以使用of去代替in。比如下面这个例子:
{% codeblock lang:html %}
  <ul id="example-1">
    <li v-for="item in items">
      {% raw %}{{ item.message }}{% endraw %}
    </li>
  </ul>
  <!-- 或者使用of代替in -->
  <ul id="example-1">
    <li v-for="item of items">
      {% raw %}{{ item.message }}{% endraw %}
    </li>
  </ul>
{% endcodeblock %}
　　渲染结果:
{% codeblock lang:html %}
  <ul>
    <li>Foo</li>
    <li>Bar</li>
  </ul>
{% endcodeblock %}
　　支持第二个参数作为当前项的索引。甚至你可以在v-for块中访问父级的属性。
{% codeblock lang:html %}
  <ul>
    <li v-for="(item, index) in items">-{% raw %}{{index}}{% endraw %} -{% raw %}{{item.message}}{% endraw %} {% raw %}{{ parentMessgae }}{% endraw %} </li>
  </ul>
{% endcodeblock %}
　　渲染结果:
{% codeblock lang:html %}
  <ul>
    <li>-0 -Foo parent</li>
    <li>-1 -Bar parent</li>
  </ul>
{% endcodeblock %}
　　除了数组之外，你也可以使用一个对象作为v-for指令的输入。比如:
{% codeblock lang:html %}
  <!-- 除了数组之外，你也可以使用对象去作为一个遍历 -->
  <p v-for="prop in forObject">
    {% raw %}{{ prop }}{% endraw %}
  </p>
{% endcodeblock %}
　　渲染结果:
{% codeblock lang:html %}
    <p>
            John
          </p>
    <p>
            Sonw
          </p>
    <p>
            40
          </p>
{% endcodeblock %}
　　与数组一样可以接受多个参数。需要注意的是，第一个位置一定是value，第二个位置一定是key，第三个是index。这与js的语言特性有关，因为其有一个arguments变量，而方法中的参数名只是针对其位置对其进行赋值而已。需要注意的是遍历的顺序是按照Object.keys()的结果，但是可能在不同的js引擎是不一致的。并且与v-if类似，vue会智能的复用组件，所以如果有需要可以在v-for遍历的组件上加上key属性去标识每一个组件提示vue不需要进行复用。
{% codeblock lang:html %}
  <p v-for="(value, key, index) in forObject" :key="index">
    {% raw %}{{key}}{% endraw %} : {% raw %}{{value}}{% endraw %} : {% raw %}{{index}}{% endraw %}
  </p>
{% endcodeblock %}
　　渲染结果:
{% codeblock lang:html %}
      <p>
              firstName : John : 0
            </p>
      <p>
              lastName : Sonw : 1
            </p>
      <p>
              age : 40 : 2
            </p>
{% endcodeblock %}
　　类似于v-if，你也可以使用template标签去包含一块需要遍历的内容。
{% codeblock lang:html %}
  <ul>
    <template v-for="item in items">
      <li>{% raw %}{{ item.msg }}{% endraw %}</li>
      <li class="divider" role="presentation"></li>
    </template>
  </ul>
{% endcodeblock %}
　　同样，如果我们不想要使用原始数组，我们也可以使用一个计算属性或者调用一个方法做到灵活运用选择。计算属性的使用与普通属性相似，下面是方法的使用:(注意，vue对数组的更新检测会由于js的限制受到对应的限制，具体查阅官方文档列表渲染部分的说明)
{% codeblock lang:js %}
  data: {
    numbers: [ 1, 2, 3, 4, 5 ]
  },
  methods: {
    even: function (numbers) {
      return numbers.filter(function (number) {
        return number % 2 === 0
      })
    }
  }
{% endcodeblock %}
{% codeblock lang:html %}
  <li v-for="n in even(numbers)">{% raw %}{{ n }}{% endraw %}</li>
{% endcodeblock %}
　　甚至可以直接取整数:
{% codeblock lang:html %}
  <div>
    <span v-for="n in 10">{% raw %}{{ n }}{% endraw %} </span>
  </div>
{% endcodeblock %}
　　渲染结果:
{% codeblock lang:html %}
 1 2 3 4 5 6 7 8 9 10
{% endcodeblock %}

### 数组更新检测
　　那么我们的列表经过渲染之后，在很多场景下可能会需要对其进行修改，这就是vue里所谓的数组更新检测。在vue中，包含一些变异方法和一些非变异方法，所谓的变异方法就是修改了原数组的内容，而非变异则是不会对原数组里的内容进行修改。但是对于vue来说，仅当变异方法的调用将会触发视图的更新。比如:
{% codeblock lang:js %}
<!-- vue包含了一些数组变异方法，通过调用这些方法我们可以更新引用该数组的视图 -->
      push()
      pop()
      shift()  // 把数组第一个元素删除掉
      unshift() // 在数组第一个位置新增一个元素
      splice()  // splice(index1, index2, ...) index1表示插入的位置，index2表示删除元素的数目，
      ... 表示新增的元素
      sort()
      reverse() ，
      当然也有一些非变异方法，比如:
      concat()、filter()、slice()，当然也可以手动去调用这些方法去为数组重新赋值
      concat() 拼凑多个数组、filter用于过滤一些不需要的值、slice相当于substring,返回起始和结束之间的索引 -->
{% endcodeblock %}
　　如果你需要使用非变异方法达到修改视图的目的，那么你需要将原来的数组指向非变异方法的返回值。
{% codeblock lang:js %}
  example1.items = example1.items.filter(function (item) {
    return item.message.match(/Foo/)
  })
{% endcodeblock %}
　　由于js的限制，不可以检测通过索引修改数组的一个项以及直接修改数组的长度的变化，为了解决第一种问题，在vue中提供了:
{% codeblock lang:js %}
  Vue.set(app.items, indexOfItem, newValue)
  // 你也可以使用 vm.$set 实例方法，该方法是全局方法 Vue.set 的一个别名：
  vm.$set(vm.items, indexOfItem, newValue)
  // 或者使用:
  // Array.prototype.splice,即变异方法
  app.items.splice(indexOfItem, 1, newValue)
{% endcodeblock %}
　　对于第二种问题:
{% codeblock lang:js %}
  vm.items.splice(newLength)
{% endcodeblock %}
　　还是由于JavaScript的限制，Vue不能检测对象属性的添加或删除，所以对于已经创建的实例，Vue不能动态添加根级别的响应式属性。比如:
{% codeblock lang:js %}
  var vm = new Vue({
    data: {
      userProfile: {
        name: 'Anika'
      }
    }
  });
{% endcodeblock %}
　　如果你想要动态的添加属性到userProfile对象中:
{% codeblock lang:js %}
  Vue.set(vm.userProfile, 'age', 27)
  // 或者别名方法:
  vm.$set(vm.userProfile, 'age', 27)
  // 如果你想要一次性添加多个属性，不可以像下面这样做
  Object.assign(vm.userProfile, {
    age: 27,
    favoriteColor: 'Vue Green'
  })
  // 应该这样
  vm.userProfile = Object.assign({}, vm.userProfile, {
    age: 27,
    favoriteColor: 'Vue Green'
  })
{% endcodeblock %}

### v-for with v-if
　　在vue的官方文档中强烈建议不要将v-if和v-for使用在同一个元素上，这样做的目的一般都只有两个:
- 过滤一个列表中的项目
- 为了避免渲染本应该被隐藏的列表

　　对于第一种目的，我们可以通过一个计算属性去达到这样的目的，或者也可以调用一个返回数组的方法。比如使用计算属性:
{% codeblock lang:html %}
  <p v-for="user in users" v-if="user.isActive"></p>
  <!-- 你可以将users改成一个计算属性activeUsers -->
  <p v-for="user in activeUsers"></p>

  computed: {
    activeUsers: function () {
      return this.users.filter(function (user) {
        return user.isActive
      })
    }
  }
{% endcodeblock %}
　　第二种，我们可以将v-if移至v-for标签的上层。比如:
{% codeblock lang:html %}
  <ul>
    <li
      v-for="user in users"
      v-if="shouldShowUsers"
      :key="user.id">
      {% raw %}{{ user.name }}{% endraw %}
    </li>
  </ul>
  <!-- 相反我们应该这样写 -->
  <ul v-if="shouldShowUsers">
    <li
      v-for="user in users"
      :key="user.id">
      {% raw %}{{ user.name }}{% endraw %}
    </li>
  </ul>
{% endcodeblock %}
　　而且，当它们处于同一节点，v-for 的优先级比 v-if 更高，这意味着 v-if 将分别重复运行于每个 v-for 循环中。所以如果我们将其放在同一个元素上面，如果我们对数据进行了修改，那么在重渲染的时候又会去遍历整个数据列表。而我们如果选择使用一个计算属性，我们在渲染的时候只遍历活跃用户，渲染更高效，而且这样解藕渲染层的逻辑，可维护性 (对逻辑的更改和扩展) 更强。

### 组件的 v-for
　　在自定义组件里，你可以像任何普通元素一样用 v-for。并且在2.2.0+ 的版本里，当在组件中使用 v-for 时，key 是必须的。
{% codeblock lang:html %}
  <my-component v-for="item in items" :key="item.id"></my-component>
{% endcodeblock %}
　　但是区别与一般的元素的是，任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域。为了把迭代数据传递到组件里，我们要用props。这样做的官方解释是明确数据来源，达到组件复用的目的。二是为了防止组件与v-for过度紧密耦合。比如:
{% codeblock lang:html %}
  <my-component
    v-for="(item, index) in items"
    v-bind:item="item"
    v-bind:index="index"
    v-bind:key="item.id">
  </my-component>
{% endcodeblock %}
　　除了key作为标识需要的属性之外，其它如item和index都是最后赋值到了props这个数组的中所定义的变量名称，然后这些变量名称将会使用在template定义的组件里。

### 小结
　　条件渲染和列表渲染有点类似于流程控制之类的东西，不过也只是简单的类比。条件和列表是一个很常用的功能，尤其是对于一些列表式的组件，但是还应该知道v-for和v-if配合使用的问题，从性能角度考虑，我们应尽量避免将v-for和v-if使用在同一个标签的上层。
