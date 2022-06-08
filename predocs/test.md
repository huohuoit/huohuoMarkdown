# Vue3 组合 API：watchEffect 与 watch

在 Vue 中，观察者用于观察 DOM 的变化或异步操作。当我们看到具有诸如单击按钮关闭导航栏或输入获取 API 的用户数据等功能的应用程序时，这可能是一个观察者在行动。在 Composition API 发布之前，`watch`在 Options API 中使用。在本文中，我们将简要了解 Composition API 并分析 Composition API 中的观察者：`watch()`函数和 `WatchEffect`。

## 什么是组合 API？

Composition API 是 Vue3 的内置功能，在最近的 Vue3.0 中推出，用于纠正 Options API 附带的`mixins`和依赖注入的错误。这个最近的特性使得使用纯 JavaScript 或 typescript 构建大型应用程序变得更加容易。它灵活，语法更简单，并且需要良好的反应性知识才能正确使用。查看[官方文档](https://vuejs.org/guide/extras/composition-api-faq.html)。

## 什么是观看选项？

此选项会监视更改，并且仅在检测到任何更改时才会运行。`Watch`需要两个参数才能运行，值有变化时运行。让我们看一个生成作者简历的简单应用程序。

```vue
<script>
export default {
  data() {
    return {
      authorBio: "My name is Namma",
    };
  },
  methods: {
    async getUser() {
      const res = await fetch("https://quotable.io/authors/?bio");
      const { results } = await res.json();
      const randomNum = Math.floor(Math.random() * results.length - 1);
      const randomItem = results[randomNum];
      this.authorBio = randomItem.bio;
    },
  },
  mounted() {
    this.getUser();
  },

  watch: {
    authorBio: (newauthorBio, oldauthorBio) => {
      console.log(oldauthorBio);
      console.log(newauthorBio);
    },
  },
};
</script>

<template>
  <p>{{ authorBio }}</p>
</template>
```

在上面的代码中，`watch`正在跟踪作者的简历，一旦检测到更改，它就会将旧值和新值记录到控制台。其他功能通过`watch Option`实现。您可以在此处阅读有关观看选项的更多信息。

现在，我们将看看较新的`watch()`函数和`WatchEffect`。

## 什么是 Watch() 函数？

该 watch 功能类似于选项，但由于 Composition APIwatch 的好处而进行了升级。函数使用`ref`作为参数。让我们使用该函数重构我们之前的代码。

Watch()

```js
// script setup
import { ref, watch, onMounted } from 'vue'

const authorBio = ref('My Name is Namma');
async function getUser() {

const res = await fetch('https://quotable.io/authors')
const {results} = await res.json()
const randomNum = Math.floor(Math.random() \* results.length-1)
const randomItem = results[randomNum]
authorBio.value = randomItem.bio
}

onMounted(() => {
getUser();
}),

watch(authorBio, (newauthorBio, oldauthorBio) => {
if (newauthorBio.startsWith('A')){
console.log("This Bio starts with A");
} else {
console.log('This Bio starts with other letters');
}

       console.log(oldauthorBio);

})
</script>
<template>

<p>{{authorBio}}</p>
</template>
```

我们在这段代码中重复我们在第一个示例中所做的事情。我们将使用该`watch()`函数操作 API 结果。我们还可以在`watch(`)函数中使用`getter`。在示例代码中，我们可以从以前的和当前的 bios 中获得一个新值。

```js
// script setup
import { ref, watch, onMounted } from 'vue'

const authorBio = ref('My Name is Namma');
const description = ref('A writer');

async function getUser() {
const res = await fetch('https://quotable.io/authors')
const {results} = await res.json()
const randomNum = Math.floor(Math.random() \* results.length-1)
const randomItem = results[randomNum]
authorBio.value = randomItem.bio
description.value = randomItem.description
}

onMounted(() => {
getUser();
}),

watch(() => authorBio.value + ' ' +description.value,(allBio) => {
console.log(allBio);

})

</script>

<template>
    <p>{{allBio}}</p>
</template>
```

Watch 将在检测到控制台更改后立即记录 allBio。

现在，让我们来看看`watchEffect`。

## 是什么 watchEffect？

`watchEffect`监视变化并执行副作用。使用时`watchEffect`，我们无法控制我们正在跟踪的依赖项。但是，`watchEffect`立即运行并观察之后的变化。让我们重构我们之前的代码。

```js
// script setup
import { ref, onMounted, watchEffect } from 'vue'

const authorBio = ref('My Name is Namma');
const description = ref('A writer');
const allBio = ref('');

async function getUser() {
const res = await fetch('https://quotable.io/authors')
const {results} = await res.json()
const randomNum = Math.floor(Math.random() \* results.length-1)
const randomItem = results[randomNum]
authorBio.value = randomItem.bio
description.value = randomItem.description
}

onMounted(() => {
getUser();
}),

watchEffect(() => {
allBio.value = (`${authorBio.value} ${description.value}`);
console.log(allBio.value);
})

</script>

<template>
    <p>{{allBio}}</p>
</template>
```

与`watch()`函数不同，它`watchEffect`跟踪不带参数的`allBio.value`并在记录更改值之前记录初始值。

## 同花顺选项

`watchEffect`如果在组件更新之前值发生变化，则立即运行并重新运行。`flush`如果我们想要应用一些更改，我们使用`post`选项，但不是立即应用。

```js
watchEffect(() => {
allBio.value = (`${authorBio.value} ${description.value}`);
console.log(allBio.value);
},
{
flush: 'post';
});
```

在 Vue3.2+ 最新版本中，有一个名为`WatchPostEffect`的新功能。

## WatchPostEffect 选项

这是 flush: post 选项的别名。如果我们希望在 Vue 更新之后而不是立即对我们的程序进行一些更改，我们将这样做：

```js
// script setup
import { ref, onMounted, watchEffect, watchPostEffect } from 'vue'

const authorBio = ref('My Name is Namma');
const description = ref('A writer');
const allBio = ref('');
const allBioId = ref(0);

async function getUser() {
const res = await fetch('https://quotable.io/authors')
const {results} = await res.json()
const randomNum = Math.floor(Math.random() \* results.length-1)
const randomItem = results[randomNum]
authorBio.value = randomItem.bio
description.value = randomItem.description
}

onMounted(() => {
getUser();
}),

watchEffect(() => {
allBio.value = (`${authorBio.value} ${description.value}`);
console.log(allBio.value);
})

watchPostEffect( () => {
allBioId.value++;
})
</script>

<template>
    <p>{{allBio}}</p>
    <p>{{allBioId}}</p>
</template>
```

这将导致`allBioId`在应用其他更改后应用更改。

## WatchSyncEffect 选项

`WatchSyncEffect`与`WatchPostEffect`一起发布，这是`flush:sync 选`项的别名。

```js
watchEffect(() => {
allBio.value = (`${authorBio.value} ${description.value}`);
console.log(allBio.value);
},
{
flush: 'sync';
})
在我们的程序中使用 WatchSyncEffect：
// script setup
import { ref, onMounted, watchEffect, watchSyncEffect } from 'vue'

const authorBio = ref('My Name is Namma');
const description = ref('A writer');
const allBio = ref('');
const allBioId = ref(0);

async function getUser() {
const res = await fetch('https://quotable.io/authors')
const {results} = await res.json()
const randomNum = Math.floor(Math.random() \* results.length-1)
const randomItem = results[randomNum]
authorBio.value = randomItem.bio
description.value = randomItem.description
}

onMounted(() => {
getUser();
}),

watchEffect(() => {
allBio.value = (`${authorBio.value} ${description.value}`);
console.log(allBio.value);
})

watchSyncEffect( () => {
allBioId.value++;
})
</script>

<template>
    <p>{{allBio}}</p>
    <p>{{allBioId}}</p>
</template>
```

`WatchSyncEffect`强制我们的程序同步。不是在单独的时间应用这些更改，而是将它们一起应用。这不是一个好的做法，因为它可能会阻碍您的程序流程。

何时使用这些观察者？
您可以为程序选择所需的功能，但您应该注意这些要点。

`watch`选项和`watch()`函数有两个值。当您想要更多地控制您正在更改的值时，您可以使用它。如果您不想立即查看更改，请使用`watch`。`Watch`懒惰地运行并在依赖项发生更改时应用更改。
如果要跟踪代码中的更改，请使用`watchEffect. watchEffect`语法比`watch`更简单，但是你不能轻易地操作依赖值。
当您想要在 Vue 组件更新后进行更改时，您可以使用`WatchPostEffect`。您可以使用`flush: post`选项代替`WatchPostEffect`；您也可以将它与`watch`。

## 结论

现在，我们已经了解了`watch`选项、`watch()`函数、`watchEffectOption API`和`Composition API`。您可以试验这些功能，直到您对它们感到满意为止。观察者还可以实现其他功能：如调试、深度观察和使观察者无效等。
