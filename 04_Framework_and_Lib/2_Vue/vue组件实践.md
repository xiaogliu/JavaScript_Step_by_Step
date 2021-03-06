# 为什么会用

假设有这样的使用场景：有个弹出框在项目的多个地方都会使用（比如各种提示框），弹出框的样式一致，只是根据不同的使用场合改变其中的文本内容，这个时候应该如何组织代码？   

当然可以在每个用到的地方复制粘贴代码，这样的方式缺点不言而喻：前期重复劳动多，后期维护不便。   

聪明的做法是将弹出框封装，然后给外界一个接口，哪里需要使用这个弹出框，只需调用这个接口就好。此时，这个封装后的弹出框就可以理解为一个子组件，而调用这个弹出框的组件便是父组件。   

# 父子组件概述

- 定义

如果在 A 组件的模版中以扩展标签的形式使用了 B 组件，则 A 组件为 B 的父组件，B 组件为 A 组件的子组件。

- 在父组件中使用子组件

这里只是简单的介绍如何能在父组件中初步使用子组件，具体的父子组件如何通信文章后半部分详细讨论。   

首先，通过 `import` 语法将子组件引入父组件中；

第二，在父组件中注册子组件。光引入还不能，还需进一步注册以后才能用，注册也简单，在父组件 Vue 实例的 `components` 属性中添加需要使用的子组件就完成了注册。   

> `components` 本身是对象，可以通过ES6对象属性简写语法。   

第三，在父组件的模版中，以标签的形式调用子组件。

> 这里要注意，按照惯例，组件在JS中采用大驼峰命名法，而在HTML中使用 kebab-case 命名法（因为HTML不区分大小写）

下面是在父组件中使用子组件的代码示例（子组件内容没有展示）   

```html
<template>
  <!-- 在父组件的模版中调用子组件 -->
  <child-component></child-component>
</template>

<script>
  // 引入子组件
  import ChildComponent from '../component_B_path/childComponent.vue';

  export default {
    data () {
      return {};
    },

    // 注册子组件
    components: {
      // 这里采用了对象属性简写的语法
      ChildComponent,
    },
  };
</script>
```

# 父组件传递信息到子组件

还是以弹出框为例。子组件的内容往往需要根据在不同地方调用改变为不同的内容，这主要由父组件决定，也就是说父组件在使用子组件时需要往子组件传递数据，并且子组件要接收接收相应数据并作出相关变化。   

- 传递数据（只传递数据，增删子元素模版中的元素）   

不同弹出框的标题会不一样，此时可通过子组件 Vue 实例的 `props` 属性接收来自父组件的数据，拿到数据后可以做进一步操作，比如改变弹出框的 title

```html
<!-- 子组件 -->
<template>
  <div class="main">
    <header>
      <!-- 这里的 title 可以通过父组件控制显示的内容 -->
      <h2>{{ title }}</h2>
    </header>
  </div>
</template>

<script>
export default {
  data () {
    return {};
  },
  props: {
    title: {
      type: String,
      default: ''
    }
  }
};
</script>
```

```html
<!-- 父组件 -->

<!-- 通过 title 往子组件传递数据“提示” -->
<ls-dialog title="提示"></ls-dialog>
```

- 内容分发（增删子元素模版中的元素）      

如果不同的弹出框里面不只是相同位置的文字不同，并且可能需要增减模版元素（比如实名认证的弹出框只需要姓名和身份证信息，但绑定银行卡的弹出框需要卡号、所属银行、办理省份，办理省份还可能会用到多级联动），如果想做通用的弹出框，只靠 `props` 传递数据进行控制就会显得力不从心。这时候内容分发就派上用场了。   

内容分发，具体讲是将父组件的内容分发到子组件，这个过程是通过在子组件中模版中定义 `slot` 来实现的。   

`slot` 可以理解为占位符，如果父组件没有往子组件分发内容，那子组件中定义在 `slot` 中的内容就会显示，不然，便会替换为父组件分发的内容。   

```html
<!-- 子组件 -->
<template>
  <div class="main">
    <main>
      <slot>如果父组件不给我分发内容，我就会显示（一般作为默认的内容）</slot>
    </main>
  </div>
</template>

<script>
// 不涉及JS代码
export default {
  data () {
    return {};
  },
};
</script>
```

```html
<!-- 父组件 -->
<template>
  <ls-dialog>
    <!-- 接下来的 p 元素会替换子组件的 slot 元素 -->
    <p>加入成功</p>
  </ls-dialog>
</template>
```

> 可以通过给 slot 元素设置 `name` 属性，这样在子元素中就可设置多个 slot，这样父元素分发内容时会更有针对性。但通过具名 slot 能实现的效果，通过单个 slot 都可以实现，并且具名插槽在可维护性上并没有绝对优势，如何选择看个人喜好了。（小白见解，随着项目经验积累或许会有更深的认识）

# 子组件传递信息到父组件

子组件往父组件传递信息是通过自定义事件完成的：**子组件每次执行发送（emit）自定义事件的代码，就相应的在父组件中触发了该自定义事件，这也就实现了父组件向子组件传递信息的目的。信息传递过来后，即自定义事件，通过在父组件调用自定义事件的事件处理程序，父组件就可以根据子组件的变化而做相关操作了**。   

下面是 Vue 

```html
<!-- 父组件 -->
<template>
  <div id="counter-event-example">
    <p>{{ total }}</p>

    <!-- increment是自定义事件，incrementTotal是在父组件中定义的事件处理程序 -->
    <child-component @child-custom-event="parentIncrementTotal()"></child-component>
    <child-component @child-custom-event="parentIncrementTotal()"></child-component>
  </div>
</template>

<script>
export default {
  data () {
    return {
      total: 0,
    };
  },
  methods: {
    // 父组件自定义事件处理程序，子组件每发送一次自定义事件，就被执行一次
    parentIncrementTotal () {
      this.total += 1;
    }
  }
};
</script>
```

```html
<!-- 子组件 -->
<template>
  <button @click="incrementCounter()">{{ counter }}</button>
</template>

<script>
export default {
  data () {
    return {
      counter: 0,
    };
  },
  method: {
    incrementCounter () {
      this.counter += 1;

      // 向父组件发送自定义事件或者说触发父组件的自定义事件
      this.$emit('child-custom-event');
    },
  },
};
</script>
```