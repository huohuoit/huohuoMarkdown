# less 语言特性

是一门`CSS`预处理，扩充了`CSS`，增加了诸如变量、混合（mixin）、函数等功能，让`CSS`更易维护、方便制作主题、扩充，`Less` 也可以运行在 Node 或浏览器端。

## 总览

变量实际上是“常量”，因为它们只能定义一次。
混合是一种将一组属性从一个规则集中包含（“混入”）到另一个规则集中的方式
嵌套规则
可以使用任何数字，颜色或变量
在使用变量和`mixin`之前不必声明它们
定义同一变量名称两次或多次后，`less`只会选择最后定义的！

## 变量

### 变量控制

变量可以从一个位置控制`CSS`规则中一些相同的值，从而使代码易于维护，同样可以在其他地方使用，如选择器名称，属性名称，`URL`和`@import`语句。

**语法：@属性名:类名;**

```css
//定义
@demo:newClass;

.@{demo}-new{ // "-new"在类名基础上，新的类名 newClass-new
   @bg:rgb(0,0,0);
   background:@bg;
}
//div使用此类名
<div class="newClass-new">
	<p>demo</p>
</div>
```

**语法：@变量名:属性名;** 常用于处理属性名太长的麻烦

```css
//定义变量（属性名）
@bg-img: background-image;

div {
  @{bg-img}: 路径;
}
```

**语法：@变量:路径;**

```css
//链接可以为任何链接*注意放置的位置
@bg-img: "路径";

header {
  background: url(@bg-img);
}
```

引入（`importing`）：你可以引入一个或者多个 less 文件，这些文件中的所有变量都可以在当前的 less 项目中使用

引入方式：`@import “main.less”`; 必须有`;`结尾，最好写上后缀名

### 使用变量名称定义变量

```css
@fnord: "I am fnord.";
@var: "fnord";
content: @@var;
// 结果
content: "I am fnord.";
```

两次定义变量时，使用变量的最后定义，从当前作用域向上搜索。这类似于`css`本身，其中`css`定义中的最后一个属性用于确定值

### 局部变量

变量写在`class`里，会变为局部变量，无法继续复用！（当`header`类执行完成后，`@color`变量会立刻销毁）

```css
header {
  // @color为局部变量，作用域只在header
  @color:rgb (255,0,0);
  background: @color;
}
div {
  // 出错，@color不存在！
  background: @color;
}
```

但是我们可以用 mixin 混合来做

**语法：.规则集名称{…}**

```css
//定义规则集
.header {
  @color:rgb (245,200,110);
  background: @color;
}

div {
  .header;
}

// 编译结果
.header {
  background: #f5c86e;
}
div {
  background: #f5c86e;
}
```

但是，如果我们不需要`css`中出现`.header`这个规则集，加个括号既可

```css
.header() {
  @color:rgb (245,200,110);
  background: @color;
}
```

## 延伸

扩展：`:extend`

### 经典用例：避免添加基类

```css
.animal {
  background-color: black;
  color: white;
}
.bear {
  &:extend(.animal);
  background-color: brown;
}
```

### 减少 CSS 大小

`Mixins`将所有属性复制到选择器中，这可能导致不必要的重复。因此，您可以使用扩展而不是混合来将选择器上移到要使用的属性，从而减少生成的`CSS`。

Example - with mixin:

```css
.my-inline-block() {
  display: inline-block;
  font-size: 0;
}
.thing1 {
  .my-inline-block;
}
.thing2 {
  .my-inline-block;
}
// 产出

.thing1 {
  display: inline-block;
  font-size: 0;
}
.thing2 {
  display: inline-block;
  font-size: 0;
}
```

示例（带有扩展）：

```css
.my-inline-block {
  display: inline-block;
  font-size: 0;
}
.thing1 {
  &:extend(.my-inline-block);
}
.thing2 {
  &:extend(.my-inline-block);
}
// 产出

.my-inline-block,
.thing1,
.thing2 {
  display: inline-block;
  font-size: 0;
}
```

## 参数混合

### 语法相关

- 参数最好用分号分隔（因为逗号可解释为`mixins`参数分隔符或`css`列表分隔符）
- 定义具有相同名称和数量的参数的多个混合是合法的
- 混合引用可以通过名称而不是位置来提供参数值。可以通过名称来引用任何参数，它们不必采用任何特殊顺序
- `@arguments`它包含调用 `mixin`时传递的所有参数，按照形参列表先后顺序
- 如果您希望`mixin`接受可变数量的参数，则可以使用`...`。在变量名之后使用此命令会将这些参数分配给变量

### 模式匹配

根据传递给它的参数来更改混合的行为

定义 `.mixin`

```css
.mixin(dark; @color) {
  color: darken(@color, 10%);
}
.mixin(light; @color) {
  color: lighten(@color, 10%);
}
.mixin(@_; @color) {
  display: block;
}
```

运行

```css
@switch: light;

.class {
  .mixin(@switch; #888);
}
```

结果

```css
.class {
  color: #a2a2a2;
  display: block;
}
```

发生了什么：

- 第一个`mixin`定义不匹配，因为它应`dark`作为第一个参数。
- 第二个`mixin`定义匹配，因为它符合预期`light`。
- 第三个`mixin`定义匹配，因为它期望任何值。

## less 注释符

在`less`中，注释符为“ // ”，与 JavaScript 等语言注释符相同。只能在`less`文件中显示，经过编译后，`css`文件里不显示！
在`less`中，如果注释符为“ /\*\*/ ”（CSS 注释符），则代表`less`文件编译成`css`文件后，在其`css`文件里显示的注释，`less`文件不显示

## 延时加载特性

- 变量不需要预先声明
- 变量可以在任意位置出现

## 父母选择器

### 多 &

代表所有父选择器（而不仅仅是最接近的祖先）

```css
.grand {
  .parent {
    & > & {
      color: red;
    }
  }
}
// 结果为
.grand .parent > .grand .parent {
  color: red;
}
```

### 更改选择器顺序

```css
.header {
  .menu {
    border-radius: 5px;
    .first & {
      background-image: url("images/button-background.png");
    }
  }
}
// 结果为
.header .menu {
  border-radius: 5px;
}
.first .header .menu {
  background-image: url("images/button-background.png");
}
```

### 组合爆炸

`&`也可以用于生成逗号分隔列表中选择器的所有可能排列

```css
p,
a,
ul,
li {
  border-top: 2px dotted #366;
  & + & {
    border-top: 0;
  }
}
// 结果
p,
a,
ul,
li {
  border-top: 2px dotted #366;
}
p + p,
p + a,
p + ul,
p + li,
a + p,
a + a,
a + ul,
a + li,
ul + p,
ul + a,
ul + ul,
ul + li,
li + p,
li + a,
li + ul,
li + li {
  border-top: 0;
}
```
