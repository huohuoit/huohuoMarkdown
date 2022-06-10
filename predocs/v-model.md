# Vue3 多 v-model 使用

按照设计，该`v-model`指令允许我们将输入值绑定到应用程序的状态。我们使用它在表单输入、文本区域和选择元素上创建双向数据绑定。它处理两个相反方向的更新：

- 当输入值发生变化时，它会将值反映到组件内部的状态上。

- 当组件状态发生变化时，它会将变化反映到表单输入元素上。

核心概念`v-model`在 Vue 3 中保持不变，但具有更多增强和功能。让我们深入挖掘！

## 视图 2：v-model

在 Vue 2 中，为了构建一个支持双向数据绑定的复杂组件，它使用单个组件`v-model`和一个完整的有效负载。

组件在内部处理所有输入元素的状态。它生成一个表示组件状态的有效负载对象。最后，它向附加了有效负载的父组件发出一个事件。

这种方法有几个缺陷，尤其是在创建 Vue UI 库时。这些陷阱中的一个是有效负载接口的模糊性。目前尚不清楚有效载荷中包含什么。开发人员必须遍历有效负载对象才能发现其中的属性。

另一个是需要在组件内部编写逻辑来处理内部状态和有效负载对象的生成。

很快，我们将发现 Vue 3 在这方面的改进。然而，在此之前，让我们回顾一下 Vue 2 如何处理在组件中实现双向数据绑定的一些基础知识。

## Vue 2：双向数据绑定

如前所述，Vue 2 使用该`v-model`指令来支持组件上的双向数据绑定。在内部，它遵循某些步骤和规则以支持该`v-model`指令。

默认情况下，该`v-model`指令使用不同的属性并为不同的输入元素发出不同的事件：

Text 和 Textarea 元素使用 value 属性，input 事件复选框和单选按钮使用 checked 属性，change 事件选择字段使用 input 属性和 change 事件。

使用单个输入元素构建组件将在内部使用类似于以下代码段的内容：

```html
<input
type="text"
:value="value"
@input="$emit('input', $event.target.value)"
```

上面的自定义组件定义了一个 prop 命名 value 如下：

```js
props: {

        value: {
            type: String,
            default: '',
            required: true
        }

}
```

然后，在父组件中，您使用新的自定义组件，如下所示：

```html
<CustomComponent v-model="name" />
```

该`v-model`指令假定`CustomComponent`定义了一个名为的内部属性 value 并发出一个名为 的事件 input。

如果`CustomComponent`必须处理多个输入怎么办？我们如何在 Vue 2 中实现这一点？

好吧，没有官方的解决方案。但是，你可以使用两种方法：

`CustomComponent`定义了一个名为 valuetype 的属性 Object。在内部，它将对象解析为 data 字段并在模板上手动进行映射。在任何字段的每次更改时，它都会为所有字段准备有效负载并发出单个 input 事件，并附加有效负载。要为这样的自定义组件编写大量代码。

另一种选择是跳过使用`v-model`指令，而是使用单独的输入/事件对。我稍后会说明这一点。

假设您有一个自定义组件来处理用户的名字和姓氏，您将使用类似的东西：

<!-- Custom Component -->

```html
<input
  type="text"
  :value="input-firstname"
  @input="$emit('input-firstname, $event.target.value)"
/>

<input
  type="text"
  :value="input-lastname"
  @input="$emit('input-lastname, $event.target.value)"
/>

<!-- and so on -->
```

至于属性，组件定义如下：

```js
props: {
  input-firstname: {
    type: String,
    required: true
  },
  input-lastname: {
    type: String,
    required: true
  },
}
```

最后，父组件使用新组件如下：

```html
<!-- parent -->

<CustomComponent
  :input-firstname="input-firstname"
  @input-firstname="input-firstname = $event"
  :input-lastname="input-lastname"
  @input-lastname="input-lastname = $event"
/>
```

我们不再使用`v-model`并在新组件上提供多个双向数据绑定。

## 视图 3：v-model

在 Vue 3 中，该`v-model`指令进行了大修，以在构建支持双向数据绑定的自定义组件时为开发人员提供更多功能和灵活性。

该`v-model`指令现在支持新的默认值。

默认`v-model`属性被重命名为，`modelValue`而不是旧名称 value。

默认`v-model`事件被重命名为，`update:modelValue` 而不是旧名称 input。

`v-model`使用`new`指令时，您可能会认为输入更多。Vue 团队领先一步，并为您提供了一种简写方式。让我们使用它重建自定义组件。

```html
<!-- Other elements in the Component -->

<input
type="text"
:value="modelValue"
@input="$emit(update:modelValue, $event.target.value)"

```

自定义组件定义单个`prop`命名`modelValue`如下：

```json
props: {

        modelValue: {
            type: String,
            default: '',
            required: true
        }

}
```

然后，在父组件中，使用新的自定义组件如下：

```html
<CustomComponent v-model:modelValue="name" />
```

新 v-model 指令提供了这样使用的新速记：

```html
<CustomComponent v-model="name" />
```

该`v-model`指令假定`CustomComponent`定义了一个名为的内部属性`modelValue`并发出一个名为 的事件`update:ModelValue`。

如果您不想使用默认命名约定，请随意使用其他名称。请记住在命名属性时要保持一致。`modelValue`下面是为属性使用自定义名称的示例。

```html
<!-- Other elements in the Component -->

<input
type="text"
:value="fullName"
@input="$emit(update:fullName, $event.target.value)"
```

上面的自定义组件定义了一个`prop`命名`modelValue`如下：

```json
props: {

        fullName: {
            type: String,
            default: '',
            required: true
        }

}
```

然后，在父组件中，您可以像这样使用新的自定义组件：

```html
<CustomComponent v-model:fullName="fullName" />
```

请注意使用属性`fullName`而不是默认属性名称。

## Vue 3：多 v-model 指令绑定

`v-model`我希望指令的 Vue 3 简写形式已经给了你一个“举手”。

这样，就 `v-model`可以灵活地 `v-model`在单个组件实例上使用多个指令。`modelValue`可以重命名为任何你想要的，让你这样做！

这个伟大的新特性消除了我之前为处理 Vue 2 的复杂自定义组件提供的两个解决方案。

让我们从构建新组件的 HTML 模板开始。

这里我们所有使用的字段都是输入元素类型。除了最后一个是复选框元素。

因此，关注一个输入字段就足够了，该字段最终将被复制到其余字段。

```html
<div class="address__field">
    <label for="address-line">Address Line 1</label>
    <input
        type="text"
        id="address-line"
        :value="addressLine"
        @input="$emit('update:addressLine', $event.target.value)”
        />
  </div>
```

输入字段根据 Vue 3 中的新指令规范`address-line`绑定`:value`指令和`@input`事件。

该组件定义了以下属性：

```js
props: {
  addressLine: {
  type: String,
  default: ""
  },
  ...
}
```

其他字段遵循相同的结构和命名约定。

让我们看一下复选框字段，看看它是如何定义的：

```html
<div class="address__field">
<label for="is-home-address">Is home address?</label>
    <input
        type="checkbox"
        id="is-home-address"
        :checked="homeAddress"
        @change="$emit('update:homeAddress', $event.target.checked)”
/>
</div>
```

在复选框字段元素的情况下，我们绑定到`:checked`指令而不是`:value`指令。此外，我们使用`@change`事件而不是`@input`在输入字段元素的情况下使用事件。事件名称遵循在新`v-model`指令中发出事件的相同标准方式。

该组件定义了以下属性：

```js
props: {
  ...
  homeAddress: {
  type: Boolean,
  default: false,
  },
}
```

现在让我们将新的自定义地址组件嵌入到`App`组件中：

```html
<b-address
  v-model:name="address.name"
  v-model:addressLine="address.addressLine"
  v-model:streetNumber="address.streetNumber"
  v-model:town="address.town"
  v-model:country="address.country"
  v-model:postcode="address.postcode"
  v-model:phoneNumber="address.phoneNumber"
  v-model:homeAddress="address.homeAddress"
/>
```

对于自定义组件上的每个属性，我们使用`v-model:{property-name}`格式进行绑定。

被`modelValue`我们手头的特定属性名称替换。当只有一个输入绑定时，速记格式就容易多了。但是，当有多个输入元素时，`modelValue`它就独树一帜了！

现在，让我们使用新的`Composition API setup()`函数定义 App 组件内的属性：

```js
setup() {
const address = reactive({
name: ""
eaddressLine: "",
streetNumber: "",
town: "",
country: "",
postcode: "",
phoneNumber: "",
homeAddress: false,
});

    return {
      address
    };

}
```

您使用对象有效负载创建一个新`reactive`属性。最后，您将响应属性返回给组件，并使用它在自定义地址组件上设置绑定，如下所示：

```bash
v-model:addressLine="address.addressLine"
```
