# 关于 Temlate 与 JSX 的选择

观察一些 Vue3 组件库的源码，我们发现，Ant Design Vue 使用了 `Options API` + `jsx/tsx` ，在`render`函数中返回`jsx/tsx`；Vant 使用了 Composition API +`jsx/tsx`，在`setup`函数中返回`jsx/tsx`；Element-Plus 使用了 `Composition API` + `template`。那么到底那个好呢，我们应该选择哪个呢？

## Options API 与 Composition API

首先，对比一下`Options API`和`Composition API`，`Composition API`在逻辑复用、类型推导、可测性等方面均占优势。而且 Vue3 都从`Options API`换成了`Composition API`，所以如何选择就不用再纠结了。

## JSX 和 Template

在 Vue 项目中，大部分场景下是推荐使用`template`的（尤其是在业务场景下）。Vue 3 基于`template`分析做了很多优化，并且对使用者是透明的，编译器默默地完成了所有的优化操作，**性价比非常高**。而使用 JSX 的话，则需要手动进行一些优化操作，比如提取静态的 JSX 片段到`render`函数外部。

### 而 Vant 为什么选择 JSX 呢？

因为 Vant 考虑到的是，组件库代码比业务代码具有更强的动态性，**使用 JSX 可以非常灵活地控制动态的 DOM 片段**。当然还可能受原先组件库的旧包袱影响。

### Element 选择 JSX/TSX

Element 的规范与 Vue3 官方推荐是一致的：优先选择`template`，当`template`写起来贼费劲时上`tsx`

> “Vue 推荐在绝大多数情况下使用模板来创建你的 HTML。然而在一些场景中，你真的需要 JavaScript 的完全编程的能力。”

### template 和 jsx/tsx 的优缺点

#### Template

- 优点：可读性高。基于`DOM`结构，很容易就可以看清楚代码要表达的意图（当然，写的一窝粥的除外）；通过 vue 的指令和语法，一眼就能确定逻辑在哪里
- 缺点：不够灵活。过于笨重`SFC`享受不到`props`类型提示；受限于`SFC`，组件在外部使用时 vscode 无法做出`props`的类型提示

#### JSX/TSX

- 优点：灵活，且可以享受到`props`的类型提
- 缺点：享受不到模板编译的优化，且可读性差。编译优化有待提高，太过于灵活容易写岔（推荐使用`render`函数）

```ts
// setup 直接返回一个函数
const Foo = {
  setup() {
    const count = ref(0);
    return () => {
      <div>{count.value}</div>;
    };
  },
};

// render 中实现
const Foo = {
  setup() {
    const count = ref(0);
    return { count };
  },
  render(ctx) {
    // 必须调用 ctx
    return <div>{ctx.count}</div>;
  },
};
```

## React 和 Vue 相关讨论

React 组件的渲染功能都依靠`JSX`。`JSX`是使用`XML`语法编写 JavaScript 的一种语法糖，有下面这些优势：

- 你可以使用完整的编程语言 JavaScript 功能来构建你的视图页面
- 开发工具对`JSX`的支持相比于现有可用的其他 Vue 模板还是比较先进的 (比如，linting、类型检查、编辑器的自动完成)

Vue 默认推荐的还是模板。任何合乎规范的`HTML`都是合法的 Vue 模板，这也带来了一些特有的优势：

- 习惯了`HTML`的开发者来说，模板比起`JSX`读写起来更自然。
- 基于`HTML`的模板应用更容易迁移框架
- 学习成本低
- 可以使用其他模板预处理器
- 修饰符带来的便利

## 到底用啥？

其实到底用啥，并没有硬性规定的，我们更应该关注使用场景！例如，你在做一个有复杂判断的生成的组件，在 js 里写`switch`会比 template 里做一堆`if`判断更加友好，做起来更舒服，这个时候就非常适合使用`jsx/tsx`。而一般的业务逻辑，相比复杂的组件没有那么多的动态性，这时候使用 template 性价比就非常高了。

## Render 函数

### render 和 template 的区别

相同：render 函数 跟 template 一样都是创建 html 模板

不同：

- Template 适合逻辑简单，render 适合复杂逻辑。
- 使用者 template 理解起来相对容易，但灵活性不足；自定义 render 函数灵活性高，但对使用者要求较高。
- render 的性能较高，template 性能较低。
- 使用 render 函数渲染没有编译过程，相当于使用者直接将代码给程序。所以，使用它对使用者要求高，且易出现错误
- Render 函数的优先级要比 template 的级别要高，但是要注意的是 Mustache(双花括号)语法就不能再次使用

先看个例子

```vue
<body>
    <div id="myDiv"></div>
</body>
<script>
const app = Vue.createApp({
  template: `
            <my-h>
                huohuo
            </my-h>
       `,
});

app.component("my-h", {
  template: `
            <h1>
                <slot />
            </h1>
        `,
});

const vm = app.mount("#myDiv");
</script>
```

如果我们需要动态改变标签（有多个）

```ts
const app = Vue.createApp({
  data() {
    return {
      myLevel: 3,
    };
  },

  template: `
            <my-h :level="myLevel">
                huohuo
            </my-h>
        `,
});

app.component("my-h", {
  props: ["level"],
  template: `
            <h1 v-if="level===1">
                <slot />
            </h1>
            <h2 v-if="level===2">
                <slot />
            </h2>
            <h3 v-if="level===3">
                <slot />
            </h3>
            <h4 v-if="level===4">
                <slot />
            </h4>
            <h5 v-if="level===5">
                <slot />
            </h5>
        `,
});
```

使用`render`函数 简化代码，动态生成 DOM

```ts
const app = Vue.createApp({
  data() {
    return {
      myLevel: 6,
    };
  },

  template: `
            <my-h :level="myLevel">
                huohuo
            </my-h>
        `,
});

app.component("my-h", {
  props: ["level"],

  render() {
    const { h } = Vue;
    return h(
      "h" + this.level,
      { name: "myh", id: "myh" },
      this.$slots.default()
    );
  },
});
```

原先的`template`改成了使用`render`方法，方法中通过`h`函数创建虚拟`DOM`节点（这个`h`函数和 Vue2.0 中`render`方法的参数`createElement`是类似的）。如果我们使用了`JSX`，那`render`方法中更可以使用 `JSX`的语法来编写虚拟`DOM`的创建。

但是我们发现这里使用了 this 对象，这跟 Composition API 提倡的函数式做法的理念并不一致。我们可以在 setup 方法中返回这个 render 方法。
