# d3-selection

`Selections` 允许强大的数据驱动文档对象模型 (DOM): 设置 [attributes](#selection_attr), [styles](#selection_style), [properties](#selection_property), [HTML](#selection_html) 或 [text](#selection_text) 内容等等。使用 [data join](#joining-data) 的 [enter](#selection_enter) 和 [exit](#selection_enter) 选择集可以用来根据具体的数据 [add](#selection_append) 或 [remove](#selection_remove) 元素。

选择集的方法通常选择当前的选择集或者新的选择集，因此允许进行链式调用。例如为所有的 `p` 元素设置类和样式:

```js
d3.selectAll("p")
    .attr("class", "graf")
    .style("color", "red");
```

等价于:

```js
var p = d3.selectAll("p");
p.attr("class", "graf");
p.style("color", "red");
```

按照惯例，返回当前选择集的方法前设置 *4* 个空格缩进，返回新的选择集的则使用 *2* 个空格缩进。这种写法有助于在链式调用中发现选择集的变化:

```js
d3.select("body")
  .append("svg")
    .attr("width", 960)
    .attr("height", 500)
  .append("g")
    .attr("transform", "translate(20,20)")
  .append("rect")
    .attr("width", 920)
    .attr("height", 460);
```

选择集是不可变的。影响被选中的元素 (或顺序) 的方法都会返回一个新的选择集，而不是在当前选择集上进行修改然后返回。但是请注意，元素必然是可变的，因为元素不变还谈什么数据驱动文档。

## Installing

NPM 安装：`npm install d3-selection`. 也可以下载 [latest release](https://github.com/d3/d3-selection/releases/latest). 此外还可以从 [d3js.org](https://d3js.org), 以 [standalone library](https://d3js.org/d3-selection.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分直接载入. 支持 AMD, CommonJS 以及最基本的标签引入形式，如果使用标签引入形式则会暴露 `d3` 全局变量:

```html
<script src="https://d3js.org/d3-selection.v1.min.js"></script>
<script>

var div = d3.selectAll("div");

</script>
```

[在浏览器中测试 d3-selection.](https://tonicdev.com/npm/d3-selection)

## API Reference

* [Selecting Elements](#selecting-elements)
* [Modifying Elements](#modifying-elements)
* [Joining Data](#joining-data)
* [Handling Events](#handling-events)
* [Control Flow](#control-flow)
* [Local Variables](#local-variables)
* [Namespaces](#namespaces)

### Selecting Elements

选择集接收 [W3C selector strings(选择字符串)](http://www.w3.org/TR/selectors-api/) 比如 `.fancy` 用来选择类名为 *fancy* 的元素， `div` 可以用来选择 *DIV* 元素。选择元素有两种模式: `select` 和 `selectAll`: `select` 只会选中第一个符合条件的元素与，而 `selectAll` 则会选中所有符合条的元素。两个顶级的选择方法: [d3.select](#select) 和 [d3.selectAll](#selectAll) 的查找范围为整个文档。两个子选择方法: [*selection*.select](#selection_select) 和 [*selection*.selectAll](#selection_selectAll) 则会将查找范围限制为*selection* 的后代元素。

<a name="selection" href="#selection">#</a> d3.<b>selection</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/index.js#L38 "Source")

选择器根元素: `document.documentElement`. 整个方法可以用来测试是否为选择集实例 (`instanceof d3.selection`) 也可以用来扩展选择集原型链，比如为 `checkbox` 添加一个 `check` 方法:

```js
d3.selection.prototype.checked = function(value) {
  return arguments.length < 1
      ? this.property("checked")
      : this.property("checked", !!value);
};
```

然后使用:

```js
d3.selectAll("input[type=checkbox]").checked(true);
```

<a name="select" href="#select">#</a> d3.<b>select</b>(<i>selector</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/select.js#L3 "Source")

选中符合条件的第一个元素，选择条件为 *selector* 字符串。如果没有元素被选中则返回空选择集。如果有多个元素都符合条件则返回包含第一个 (文档顺序) 匹配的元素的选择集。例如选择第一个 `a` 元素:

```js
var anchor = d3.select("a");
```

*selector* 除是字符串之外还可以是指定的节点，在已经对节点有引用的时候是很有用的。比如在事件监听器回调内 `this` 指向节点时就可以使用这种方法将其转换为选择集实例:

```js
d3.selectAll("p").on("click", function() {
  d3.select(this).style("color", "red");
});
```

<a name="selectAll" href="#selectAll">#</a> d3.<b>selectAll</b>(<i>selector</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selectAll.js#L3 "Source")

选择所有与指定的 *selector* 匹配的元素。被选中的元素顺序会按其在文档中的顺序排列(从上到下)。如果没有元素被选中，或者 *selector* 为 null 或 undefined 则返回空选择集。例如选择所有的 `p` 元素：

```js
var paragraph = d3.selectAll("p");
```

*selector* 除字符串之外还可以是节点数组。如果已经对节点有引用时会很有用，比如在事件回调内将 `this.childNodes` 或 `document.links` 传递给选择器。节点可以是一个伪数组，比如`NodeList` 或 `arguments`，比如将所有链接的颜色改为红色:

```js
d3.selectAll(document.links).style("color", "red");
```

<a name="selection_select" href="#selection_select">#</a> <i>selection</i>.<b>select</b>(<i>selector</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/select.js "Source")

对于每个已经选中的元素，进一步选择它们的第一个匹配的后代元素。如果没有元素匹配的元素被选中，则返回的选择集中当前索引处的元素为 `null`. 如果 `selector` 为 `null` 则返回的选择集中的每个元素都为 `null`，也就是产生一个空选择集。如果当前的元素有关联数据，则数据将被传递给对应的被选中的元素。如果有多个元素匹配当前的选择条件则返回第一个符合条件的元素。例如选中文档中所有的 `p` 元素下的第一个 `b` 元素:

```js
var b = d3.selectAll("p").select("b");
```

如果 *selector* 为函数，则会在选择前执行对应的函数，并且会传递当前元素的关联数据 (*d*)，当前的索引 (*i*) 以及当前分组 (*nodes*)，在函数中 *this* 指向当前 `DOM` 元素(*nodes*[*i*]). 为函数时必须返回一个元素，或者 `null`。例如选择每个 `p` 元素的前一个同胞节点:

```js
var previous = d3.selectAll("p").select(function() {
  return this.previousElementSibling;
});
```

与 [*selection*.selectAll](#selection_selectAll) 不同, *selection*.select 不会影响分组: 它会保存现有的分组结构以及索引并且将关联数据(如果有的话)传递给选中的子节点。分组在
[data join](#joining-data) 中扮演着很重要的角色。参考 [Nested Selections](http://bost.ocks.org/mike/nest/) 和 [How Selections Work](http://bost.ocks.org/mike/selection/) 获取更多信息.

<a name="selection_selectAll" href="#selection_selectAll">#</a> <i>selection</i>.<b>selectAll</b>(<i>selector</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/selectAll.js "Source")

对于已经选中的元素，进一步选择其所有符合条件的后代元素。返回的元素会按照其父节点进行分组。对于某个元素来说，如果没有符合条件的后代元素，则当前索引处的组将为 `null`. 选中的元素不会继承其父选择集的数据。可以使用 [*selection*.data](#selection_data) 操作将其传递给子元素。例如选择每个 `p` 标签中的所有的 `b` 元素:

```js
var b = d3.selectAll("p").selectAll("b");
```

如果 *selector* 是一个函数，则在进一步选择前会先执行，并依次传递当前的数据 *d*，当前的索引 (*i*) 以及当前的分组 (*nodes*), 在函数内部 *this* 指向当前的 DOM 元素(*nodes*[*i*]), 返回值必须是一个元素数组(或者类数组，比如 NodeList)，也可以为空数组(如果没有符合条件的元素)。例如选择所有 `p` 元素的前一个同胞节点和后一个同胞节点:

```js
var sibling = d3.selectAll("p").selectAll(function() {
  return [
    this.previousElementSibling,
    this.nextElementSibling
  ];
});
```

与 [*selection*.select](#selection_select) 不同, *selection*.selectAll 会影响分组: 每个选中的后代元素根据其父元素进行分组。分组在 [data join](#joining-data) 中扮演着很重要的角色。参考 [Nested Selections](http://bost.ocks.org/mike/nest/) 和 [How Selections Work](http://bost.ocks.org/mike/selection/) 获取更多信息。

<a name="selection_filter" href="#selection_filter">#</a> <i>selection</i>.<b>filter</b>(<i>filter</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/filter.js "Source")

过滤选择集并返回一个新的过滤后的选择集。*filter* 可以是一个选择字符串也可以是一个函数。如果 *filter* 为函数则会为被过滤的选择集中的每个元素进行调用，并传递当前的数据 (*d*), 当前的索引 (*i*) 以及当前的分组 (*nodes*), 函数中 *this* 指向当前 DOM 元素 (*nodes*[*i*]).

例如，过滤出表格行中的偶数行:

```js
var even = d3.selectAll("tr").filter(":nth-child(even)");
```

这与直接使用 [d3.selectAll](#selectAll) 的效果近似等价，只不过索引可能不一样:

```js
var even = d3.selectAll("tr:nth-child(even)");
```

用函数实现:

```js
var even = d3.selectAll("tr").filter(function(d, i) { return i & 1; });
```

或者使用 [*selection*.select](#selection_select):

```js
var even = d3.selectAll("tr").select(function(d, i) { return i & 1 ? this : null; });
```

要注意的是 `:nth-child` 伪类基于 1 的索引而不是基于 0 开始。此外上述过滤函数与 `:nth-child`
的含义完全不同，一个是基于索引而另一个是基于 DOM 中兄弟元素的数量。

返回的经过过滤的选择集保存了选择集的父元素，但与 [*array*.filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) 类似不会保留索引，因为在过滤中可能会删除某些元素，如果需要可以使用 [*selection*.select](#selection_select) 来保留索引。

<a name="selection_merge" href="#selection_merge">#</a> <i>selection</i>.<b>merge</b>(<i>other</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/merge.js "Source")

返回一个将当前选择集和指定的 *other* 选择集合并之后的新的选择集。返回的选择集与当前选择集有相同数量的分组以及相同的父元素。当前选择集中的任何空缺 (`null`) 都会被指定的 *other* 中的对应的元素填充(如果存在的话)，如果 *other* 选择集有附加的组或者父元素，则将其忽略。

这个方法通常用来在 [data-join](#joining-data) 操作之后合并 [enter](#selection_enter) 和 [update](#selection_data)。在单独修改输入和更新元素之后对其进行合并执行其他的操作而不需要额外的代码。例如:

```js
var circle = svg.selectAll("circle").data(data) // UPDATE
    .style("fill", "blue");

circle.exit().remove(); // EXIT

circle = circle.enter().append("circle") // ENTER
    .style("fill", "green")
  .merge(circle) // ENTER + UPDATE
    .style("stroke", "black");
```

参考 [*selection*.data](#selection_data) 获取更多信息，这也是更新 DOM 的一般形式。

但是这个方法不适用于合并任意选择集: 如果原选择集与指定的 *other* 选择集在同样的索引下都有元素，则原选择集中的元素会被返回而 *other* 选择集中的元素会被忽略。

<a name="matcher" href="#matcher">#</a> d3.<b>matcher</b>(<i>selector</i>) [<>源码](https://github.com/d3/d3-selection/blob/master/src/matcher.js "Source")

根据指定的 *selector* 返回一个用来判断 *this* 元素是否匹配此选择器的方法，这个方法在 [*selection*.filter](#selection_filter) 内部使用，例如:

```js
var div = selection.filter("div");
```

等价于:

```js
var div = selection.filter(d3.matcher("div"));
```

(尽管 D3 不是一个做兼容的层，虽然 *element*.matches 标准的不断变化，但是 D3 内部还是会做前缀处理)

<a name="selector" href="#selector">#</a> d3.<b>selector</b>(<i>selector</i>) [<>源码](https://github.com/d3/d3-selection/blob/master/src/selector.js "Source")

根据指定的 *selector* 返回一个函数，这个函数返回 `this` 元素的第一个符合条件的后代元素。内部通过 [*selection*.select](#selection_select) 实现。例如:

```js
var div = selection.select("div");
```

等价于:

```js
var div = selection.select(d3.selector("div"));
```

<a name="selectorAll" href="#selectorAll">#</a> d3.<b>selectorAll</b>(<i>selector</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selectAll.js "Source")

根据指定的 *selector* 返回一个函数，这个函数返回 `this` 元素的所有符合条件的后代元素。内部通过 [*selection*.selectAll](#selection_selectAll) 实现。例如:

```js
var div = selection.selectAll("div");
```

等价于:

```js
var div = selection.selectAll(d3.selectorAll("div"));
```

<a name="window" href="#window">#</a> d3.<b>window</b>(<i>node</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/window.js "Source")

返回指定的 *node* 所属的 `window`。如果 *node* 为 `Node` 则返回所属文档的默认视图,如果 *node* 为 `document` 则返回其默认视图, 其他情况返回 *node*。

<a name="style" href="#style">#</a> d3.<b>style</b>(<i>node</i>, <i>name</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/style.js#L32 "Source")

返回指定 *node* 的 *name* 对应的 `style` 属性值. 如果 *node* 对应的 *name* 样式为行内样式，则返回其值。否则返回 [computed property value](https://developer.mozilla.org/en-US/docs/Web/CSS/computed_value)，参考 [*selection*.style](#selection_style).

### Modifying Elements

在选中元素之后，使用选择集的转换方法来修改文档的内容。例如设置 `a` 标签的 `name` 属性和 `color` 样式:

```js
d3.select("a")
    .attr("name", "fred")
    .style("color", "red");
```

可以访问 [d3js.org](https://d3js.org) 并且打开控制台来试验选择集的相关操作. 比如查看操作选择集或者调用选择集的某些方法会影响哪些页面内容。

<a name="selection_attr" href="#selection_attr">#</a> <i>selection</i>.<b>attr</b>(<i>name</i>[, <i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/attr.js "Source")

如果指定了 *value* 则将指定 *name* 的属性值设置为指定的 *value* 并返回当前选择集。如果 *value* 为常量则会将选择集中的每个元素的 *name* 属性设置为该值。如果 *value* 为函数则会为选择集中的每个元素进行调用，并传递当前绑定的数据 *d* , 索引 *i* 以及当前组 *nodes*，函数内部 *this* 指向当前 DOM 元素(*nodes*[*i*]). 函数的返回值将会被设置为对应属性的值。如果返回 `null` 则会移除当前 *name* 属性。

如果没有指定 *value* 则返回当前选择集中第一个非空元素的对应的属性值。当已知选择集中只包含一个元素时，使用这个方法是很有用的。

指定的 *name* 可以指定一个命名空间前缀，比如 `xlink:href` 指定 `href` 属性为 `XLink` 命名空间. 参考 [namespaces](#namespaces) 获取更多关于命名空间的信息。可以通过映射来注册其他命名空间。

<a name="selection_classed" href="#selection_classed">#</a> <i>selection</i>.<b>classed</b>(<i>names</i>[, <i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/classed.js "Source")

如果指定了 *value* 则通过设置 `class` 属性或者修改 `classList` 来为选择集中的元素指定或取消指定名称为 *names* 的类名。*names* 可以是由空格分割的多个类名的组合，比如为元素指定  `foo` 和 `bar` 两个类名:

```js
selection.classed("foo bar", true);
```

如果 *value* 为真，则表示选择集中的所有元素都会被添加指定的类名。否则则指定的类会被移除。如果 *value* 为函数，则会为选择集中的每个元素调用，并传递当前元素绑定的数据 *d*, 索引 *i* 以及当前的分组 *nodes*。在函数内部 `this` 指向当前元素(*nodes*[*i*]). 函数的返回值决定是否启用指定的类名。例如随机的为每个元素启用或者移除名为 *foo* 的类名:

```js
selection.classed("foo", function() { return Math.random() > 0.5; });
```

如果没有指定 *value* 则返回当前选择集中第一个非空元素是否启用了指定的 *classes*. 在已知选择集中只包含一个元素时很有用。

<a name="selection_style" href="#selection_style">#</a> <i>selection</i>.<b>style</b>(<i>name</i>[, <i>value</i>[, <i>priority</i>]]) [<>源码](https://github.com/d3/d3-selection/blob/master/src/selection/style.js "Source")

如果指定了 *value* 则将指定 *name* 的样式设置为指定的 *value* 并返回当前选择集。如果 *value* 为常量则会将选择集中的每个元素的 *name* 样式设置为该值。如果 *value* 为函数则会为选择集中的每个元素进行调用，并传递当前绑定的数据 *d* , 索引 *i* 以及当前组 *nodes*，函数内部 *this* 指向当前 DOM 元素(*nodes*[*i*]). 函数的返回值将会被设置为对应样式的值。如果返回 `null` 则会移除当前 *name* 属性。可选的 *priority*(优先级) 可以为 null 也可以为 `important`(没有感叹号).

如果没有指定 *value* 则返回当前选择集中第一个非空元素对应的样式。当前样式为元素的内联样式值(如果存在), 否则会返回其 [computed value(计算值)](https://developer.mozilla.org/en-US/docs/Web/CSS/computed_value)。当已知选择集中只包含一个元素时很有用。

警告：与大多数 SVG 属性不同，CSS 样式通常具有关联的单位。比如 `3px` 是有效的 stroke-width 属性值，而 `3` 则不是。一些浏览器会隐式添加 `px` 单位，但是不是全部浏览器都会这样，比如 IE 会抛出 “invalid arguments” (无效参数) 错误。

<a name="selection_property" href="#selection_property">#</a> <i>selection</i>.<b>property</b>(<i>name</i>[, <i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/property.js "Source")

有些 HTML 元素的属性比较特殊，不能直接使用 `attr` 和 `style` 操作，比如文本域的 `value` 属性以及 checkbox 的 `checked` 属性。使用本方法可以操作这些属性。

如果指定了 *value* 则将选中的元素对应 *name* 属性值设置为指定的 *value*。如果 *value* 为常量，则将选择集中所有的元素对应的属性都设置为指定的 *value* 常量。如果 *value* 为函数，则会为选择集中的所有元素依次调用，并传递当前元素绑定的数据 *d*，当前的索引 *i* 以及当前分组 *nodes*，在函数内部 *this* 指向当前 `DOM` 元素(*nodes*[*i*])。函数的返回值将会被设置为当前元素的属性值。使用 `null` 作为值表示移除当前属性。

如果没有指定 *value* 则返回当前选择集中第一个非空元素对应的属性值。当已知选择集中只包含一个元素时很有用。

<a name="selection_text" href="#selection_text">#</a> <i>selection</i>.<b>text</b>([<i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/text.js "Source")

如果指定了 *value* 则将选中的元素的 [text content](http://www.w3.org/TR/DOM-Level-3-Core/core.html#Node3-textContent) 设置为指定的值，会替代任何现有的子元素。如果 *value* 为常量则将当前选择集中的所有元素都设置为相同的文本内容。如果 *value* 为函数则会为选择集中的所有元素依次调用，并传递当前元素绑定的数据 *d*，当前的索引 *i* 以及当前分组 *nodes*，在函数内部 *this* 指向当前 `DOM` 元素(*nodes*[*i*])。函数的返回值会被设置为当前元素的文本内容。使用 `null` 作为值表示移除当前文本内容。

如果没有指定 *value* 则返回当前选择集中第一个非空元素的文本内容。当已知选择集中只包含一个元素时很有用。

<a name="selection_html" href="#selection_html">#</a> <i>selection</i>.<b>html</b>([<i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/html.js "Source")

如果指定了 *value* 则将选中的元素的 [inner HTML](http://dev.w3.org/html5/spec-LC/apis-in-html-documents.html#innerhtml) 设置为指定的值，会替代任何现有的子元素。如果 *value* 为常量则将当前选择集中的所有元素都设置为相同的 `HTML` 内容。如果 *value* 为函数则会为选择集中的所有元素依次调用，并传递当前元素绑定的数据 *d*，当前的索引 *i* 以及当前分组 *nodes*，在函数内部 *this* 指向当前 `DOM` 元素(*nodes*[*i*])。函数的返回值会被设置为当前元素的 `HTML` 内容。使用 `null` 作为值表示移除当前 `HTML` 内容。

如果没有指定 *value* 则返回当前选择集中第一个非空元素的 `HTML` 内容。当已知选择集中只包含一个元素时很有用。

使用 [*selection*.append](#selection_append) 或 [*selection*.insert](#selection_insert) ；这个方法适用于需要很少的 HTML 内容，常用来丰富格式。*selection*.html 仅仅支持 HTML 元素，SVG 元素以及其他的非 HTML 元素不支持 `innerHTML` 属性因此不能使用 *selection*.html 方法。考虑使用 [XMLSerializer](https://developer.mozilla.org/en-US/docs/XMLSerializer) 将 DOM 树转换为文本。[innersvg polyfill](https://code.google.com/p/innersvg/) 提供了一个能在 SVG 元素上使用 innerHTML 属性的垫片。

<a name="selection_append" href="#selection_append">#</a> <i>selection</i>.<b>append</b>(<i>type</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/append.js "Source")

如果指定的 *type* 为字符串则创建一个以此字符串为标签名的元素，并将其追加到选择集元素列表中。如果选择集为 [enter selection](#selection_enter) 则将其添加到下一个同胞节点之前。后续的 enter 选择集将和新的元素一起被插入到 DOM 中。但是要注意的是当绑定的数据发生顺序变化时仍然需要使用 [*selection*.order](#selection_order) 来同步更新元素的次序。(比如新元素与之前绑定的数据元素之间的次序发生变化)

如果 *type* 为函数则会为每个选中的元素执行，并传递当前绑定的元素 *d*，当前索引 *i* 以及当前分组 *nodes*，函数内部 *this* 指向当前 DOM 元素(*nodes*[*i*]). 函数应该返回一个元素用来被添加到 DOM 中(通常在函数内部创建一个新元素节点返回，但是也可能会返回一个已有的元素)。例如为每个 `p` 标签中添加一个 `div` 元素:

```js
d3.selectAll("p").append("div");
```

等价于:

```js
d3.selectAll("p").append(function() {
  return document.createElement("div");
});
```

等价于:

```js
d3.selectAll("p").select(function() {
  return this.appendChild(document.createElement("div"));
});
```

无论是 `type` 是字符串还是返回 DOM 元素的函数，都会返回一个新的包含被添加元素的选择集。每个新的元素都会继承当前元素的数据(如果有的话)，参考 [*selection*.select](#selection_select)。

指定的标签 *name* 可能包含一个命名空间前缀，比如使用 `svg:text` 指定 SVG 命名空间下的 `text` 元素。参考 [namespaces](#namespaces) 获取支持的命名空间，也可以注册命名空间。如果没有指定命名空间则会继承父元素的命名空间；如果指定的标签名的命名空间是已知的，则对应的命名空间会被使用(比如 `svg` 暗指 `svg:svg`).

<a name="selection_insert" href="#selection_insert">#</a> <i>selection</i>.<b>insert</b>(<i>type</i>[, <i>before</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/insert.js "Source")

如果 *type* 为字符串则为选择集中每个选中的插入一个指定类型(标签名)的元素，插入的位置为第一个匹配 *before* 选择条件的元素。例如使用 `:first-child` 会将新的元素插入到第一个子元素的位置。如果没有指定 *before* 则默认为 `null`。(按 [bound data(数据绑定)](#joining-data) 次序添加元素考虑使用 [*selection*.append](#selection_append).)

*type* 和 *before* 都可以使用函数代替，函数会为选择集中的每个元素调用，并传递当前元素绑定的数据 *d*，索引 *i* 以及当前分组 *nodes*，函数内部 *this* 指向当前的 DOM 元素(*nodes*[*i*]). *type* 函数应该返回一个被插入的元素，*before* 函数有应该返回当前元素的子元素用来定位被插入元素的位置。例如为每个 `p` 元素插入 `DIV` 元素：

```js
d3.selectAll("p").insert("div");
```

等价于:

```js
d3.selectAll("p").insert(function() {
  return document.createElement("div");
});
```

等价于：

```js
d3.selectAll("p").select(function() {
  return this.insertBefore(document.createElement("div"), null);
});
```

在上述两种例子中，都会返回包含被插入元素的新的选择集。每个新元素继承了当前元素的数据(如果有的话)，继承方式与 [*selection*.select](#selection_select) 相同。

指定的标签 *name* 可能包含一个命名空间前缀，比如使用 `svg:text` 指定 SVG 命名空间下的 `text` 元素。参考 [namespaces](#namespaces) 获取支持的命名空间，也可以注册命名空间。如果没有指定命名空间则会继承父元素的命名空间；如果指定的标签名的命名空间是已知的，则对应的命名空间会被使用(比如 `svg` 暗指 `svg:svg`).

<a name="selection_remove" href="#selection_remove">#</a> <i>selection</i>.<b>remove</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/remove.js "Source")

从当前文档中移除选中的元素。返回的选择集(被移除的元素)已经与文档脱离。目前还没有专门的 API 将删除的元素重新填加到文档中，但是你可以通过 [*selection*.append](#selection_append) 或 [*selection*.insert](#selection_insert) 将其重新加回文档。

<a name="selection_clone" href="#selection_clone">#</a> <i>selection</i>.<b>clone</b>([<i>deep</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/clone.js "Source")

在所选元素之后插入所选元素的克隆，并返回包含新添加的克隆元素的选择集。如果 *deep* 为真则选中元素的后代元素也会被克隆(深度克隆)。否则仅仅克隆所选元素自身。等价于：

```js
selection.select(function() {
  return this.parentNode.insertBefore(this.cloneNode(deep), this.nextSibling);
});
```

<a name="selection_sort" href="#selection_sort">#</a> <i>selection</i>.<b>sort</b>(<i>compare</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/sort.js "Source")

返回一个新选择集，其中包含了当前选择集中所有元素的经过 *compare* 函数排序之后的副本。在排序后，将排序后的元素重新插入原来的文档中替换未排序的元素。([*selection*.order](#selection_order))

比较函数默认为 [ascending](https://github.com/d3/d3-array#ascending)，会传递两个元素绑定的数据 *a* 和 *b*。函数返回负值，正值或者 0. 如果是负值则表示 *a* 应该在 *b* 之前，如果是正值则表示 *a* 应该在 *b* 之后，否则不调整 *a* 和 *b* 的位置。

要注意的是，排序不能保证稳定；但是能保证与浏览器内置的数组 [sort](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array/sort) 方法有相同的行为。

<a name="selection_order" href="#selection_order">#</a> <i>selection</i>.<b>order</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/order.js "Source")

Re-inserts elements into the document such that the document order of each group matches the selection order. This is equivalent to calling [*selection*.sort](#selection_sort) if the data is already sorted, but much faster.

<a name="selection_raise" href="#selection_raise">#</a> <i>selection</i>.<b>raise</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/raise.js "Source")

Re-inserts each selected element, in order, as the last child of its parent. Equivalent to:

```js
selection.each(function() {
  this.parentNode.appendChild(this);
});
```

<a name="selection_lower" href="#selection_lower">#</a> <i>selection</i>.<b>lower</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/lower.js "Source")

Re-inserts each selected element, in order, as the first child of its parent. Equivalent to:

```js
selection.each(function() {
  this.parentNode.insertBefore(this, this.parentNode.firstChild);
});
```

<a name="create" href="#create">#</a> d3.<b>create</b>(<i>name</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/create.js "Source")

Given the specified element *name*, returns a single-element selection containing a detached element of the given name in the current document.

<a name="creator" href="#creator">#</a> d3.<b>creator</b>(<i>name</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/creator.js "Source")

Given the specified element *name*, returns a function which creates an element of the given name, assuming that `this` is the parent element. This method is used internally by [*selection*.append](#selection_append) and [*selection*.insert](#selection_insert) to create new elements. For example, this:

```js
selection.append("div");
```

Is equivalent to:

```js
selection.append(d3.creator("div"));
```

See [namespace](#namespace) for details on supported namespace prefixes, such as for SVG elements.

### Joining Data

For an introduction to D3’s data joins, see [Thinking With Joins](http://bost.ocks.org/mike/join/). Also see the [General Update Pattern](http://bl.ocks.org/mbostock/3808218) examples.

<a name="selection_data" href="#selection_data">#</a> <i>selection</i>.<b>data</b>([<i>data</i>[, <i>key</i>]]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/data.js "Source")

Joins the specified array of *data* with the selected elements, returning a new selection that represents the *update* selection: the elements successfully bound to data. Also defines the [enter](#selection_enter) and [exit](#selection_exit) selections on the returned selection, which can be used to add or remove elements to correspond to the new data. The specified *data* is an array of arbitrary values (*e.g.*, numbers or objects), or a function that returns an array of values for each group. When data is assigned to an element, it is stored in the property `__data__`, thus making the data “sticky” and available on re-selection.

The *data* is specified **for each group** in the selection. If the selection has multiple groups (such as [d3.selectAll](#selectAll) followed by [*selection*.selectAll](#selection_selectAll)), then *data* should typically be specified as a function. This function will be evaluated for each group in order, being passed the group’s parent datum (*d*, which may be undefined), the group index (*i*), and the selection’s parent nodes (*nodes*), with *this* as the group’s parent element. For example, to create an HTML table from a matrix of numbers:

```js
var matrix = [
  [11975,  5871, 8916, 2868],
  [ 1951, 10048, 2060, 6171],
  [ 8010, 16145, 8090, 8045],
  [ 1013,   990,  940, 6907]
];

var tr = d3.select("body")
  .append("table")
  .selectAll("tr")
  .data(matrix)
  .enter().append("tr");

var td = tr.selectAll("td")
  .data(function(d) { return d; })
  .enter().append("td")
    .text(function(d) { return d; });
```

In this example the *data* function is the identity function: for each table row, it returns the corresponding row from the data matrix.

If a *key* function is not specified, then the first datum in *data* is assigned to the first selected element, the second datum to the second selected element, and so on. A *key* function may be specified to control which datum is assigned to which element, replacing the default join-by-index, by computing a string identifier for each datum and element. This key function is evaluated for each selected element, in order, being passed the current datum (*d*), the current index (*i*), and the current group (*nodes*), with *this* as the current DOM element (*nodes*[*i*]); the returned string is the element’s key. The key function is then also evaluated for each new datum in *data*, being passed the current datum (*d*), the current index (*i*), and the group’s new *data*, with *this* as the group’s parent DOM element; the returned string is the datum’s key. The datum for a given key is assigned to the element with the matching key. If multiple elements have the same key, the duplicate elements are put into the exit selection; if multiple data have the same key, the duplicate data are put into the enter selection.

For example, given this document:

```html
<div id="Ford"></div>
<div id="Jarrah"></div>
<div id="Kwon"></div>
<div id="Locke"></div>
<div id="Reyes"></div>
<div id="Shephard"></div>
```

You could join data by key as follows:


```js
var data = [
  {name: "Locke", number: 4},
  {name: "Reyes", number: 8},
  {name: "Ford", number: 15},
  {name: "Jarrah", number: 16},
  {name: "Shephard", number: 31},
  {name: "Kwon", number: 34}
];

d3.selectAll("div")
  .data(data, function(d) { return d ? d.name : this.id; })
    .text(function(d) { return d.number; });
```

This example key function uses the datum *d* if present, and otherwise falls back to the element’s id property. Since these elements were not previously bound to data, the datum *d* is null when the key function is evaluated on selected elements, and non-null when the key function is evaluated on the new data.

The *update* and *enter* selections are returned in data order, while the *exit* selection preserves the selection order prior to the join. If a key function is specified, the order of elements in the selection may not match their order in the document; use [*selection*.order](#order) or [*selection*.sort](#sort) as needed. For more on how the key function affects the join, see [A Bar Chart, Part 2](http://bost.ocks.org/mike/bar/2/) and [Object Constancy](http://bost.ocks.org/mike/constancy/).

Although the data-join can be used simply to create (to *enter*) a set of elements corresponding to data, more generally the data-join is designed to let you create, destroy or update elements as needed so that the resulting DOM corresponds to the new data. The data-join lets you do this efficiently by executing only the minimum necessary operations on each state of element (entering, updating, or exiting), and allows you to declare concise animated transitions between states as well. Here is a simple example of the [General Update Pattern](http://bl.ocks.org/mbostock/3808218):

```js
var circle = svg.selectAll("circle") // 1
  .data(data) // 2
    .style("fill", "blue"); // 3

circle.exit().remove(); // 4

circle = circle.enter().append("circle") // 5, 9
    .style("fill", "green") // 6
  .merge(circle) // 7
    .style("stroke", "black"); // 8
```

Breaking this down into discrete steps:

1. Any existing circles (that are descendants of the `svg` selection) are [selected](#selection_selectAll).
2. These circles are [joined to new `data`](#selection_data), returning the matching circles: the *update* selection.
3. These updating circles are given a blue fill.
4. Any existing circles that do *not* match new data—the *exit* selection—are removed.
5. New circles are [appended](#selection_append) for any new data that do *not* match any existing circle: the *enter* selection.
6. These entering circles are given a green fill.
7. A new selection representing the [union](#selection_merge) of entering and updating circles is created.
8. These entering and updating circles are given a black stroke.
9. These circles are stored in the variable `circle`.

As described in the preceding paragraphs, the “matching” logic is determined by the key function passed to *selection*.data; since no key function is used in the above code sample, the elements and data are joined by index.

If *data* is not specified, this method returns the array of data for the selected elements.

This method cannot be used to clear bound data; use [*selection*.datum](#selection_datum) instead.

<a name="selection_enter" href="#selection_enter">#</a> <i>selection</i>.<b>enter</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/enter.js "Source")

Returns the enter selection: placeholder nodes for each datum that had no corresponding DOM element in the selection. (The enter selection is empty for selections not returned by [*selection*.data](#selection_data).)

The enter selection is typically used to create “missing” elements corresponding to new data. For example, to create DIV elements from an array of numbers:

```js
var div = d3.select("body")
  .selectAll("div")
  .data([4, 8, 15, 16, 23, 42])
  .enter().append("div")
    .text(function(d) { return d; });
```

If the body is initially empty, the above code will create six new DIV elements, append them to the body in-order, and assign their text content as the associated (string-coerced) number:

```html
<div>4</div>
<div>8</div>
<div>15</div>
<div>16</div>
<div>23</div>
<div>42</div>
```

Conceptually, the enter selection’s placeholders are pointers to the parent element (in this example, the document body). The enter selection is typically only used transiently to append elements, and is often [merged](#selection_merge) with the update selection after appending, such that modifications can be applied to both entering and updating elements.

<a name="selection_exit" href="#selection_exit">#</a> <i>selection</i>.<b>exit</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/exit.js "Source")

Returns the exit selection: existing DOM elements in the selection for which no new datum was found. (The exit selection is empty for selections not returned by [*selection*.data](#selection_data).)

The exit selection is typically used to remove “superfluous” elements corresponding to old data. For example, to update the DIV elements created previously with a new array of numbers:

```js
div = div.data([1, 2, 4, 8, 16, 32], function(d) { return d; });
```

Since a key function was specified (as the identity function), and the new data contains the numbers [4, 8, 16] which match existing elements in the document, the update selection contains three DIV elements. Leaving those elements as-is, we can append new elements for [1, 2, 32] using the enter selection:

```js
div.enter().append("div").text(function(d) { return d; });
```

Likewise, to remove the exiting elements [15, 23, 42]:

```js
div.exit().remove();
```

Now the document body looks like this:

```html
<div>1</div>
<div>2</div>
<div>4</div>
<div>8</div>
<div>16</div>
<div>32</div>
```

The order of the DOM elements matches the order of the data because the old data’s order and the new data’s order were consistent. If the new data’s order is different, use [*selection*.order](#selection_order) to reorder the elements in the DOM. See the [General Update Pattern](http://bl.ocks.org/mbostock/3808218) example thread for more on data joins.

<a name="selection_datum" href="#selection_datum">#</a> <i>selection</i>.<b>datum</b>([<i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/datum.js "Source")

Gets or sets the bound data for each selected element. Unlike [*selection*.data](#selection_data), this method does not compute a join and does not affect indexes or the enter and exit selections.

If a *value* is specified, sets the element’s bound data to the specified value on all selected elements. If the *value* is a constant, all elements are given the same datum; otherwise, if the *value* is a function, it is evaluated for each selected element, in order, being passed the current datum (*d*), the current index (*i*), and the current group (*nodes*), with *this* as the current DOM element (*nodes*[*i*]). The function is then used to set each element’s new data. A null value will delete the bound data.

If a *value* is not specified, returns the bound datum for the first (non-null) element in the selection. This is generally useful only if you know the selection contains exactly one element.

This method is useful for accessing HTML5 [custom data attributes](http://www.w3.org/TR/html5/dom.html#custom-data-attribute). For example, given the following elements:

```html
<ul id="list">
  <li data-username="shawnbot">Shawn Allen</li>
  <li data-username="mbostock">Mike Bostock</li>
</ul>
```

You can expose the custom data attributes by setting each element’s data as the built-in [dataset](http://www.w3.org/TR/html5/dom.html#dom-dataset) property:

```js
selection.datum(function() { return this.dataset; })
```

### Handling Events

For interaction, selections allow listening for and dispatching of events.

<a name="selection_on" href="#selection_on">#</a> <i>selection</i>.<b>on</b>(<i>typenames</i>[, <i>listener</i>[, <i>capture</i>]]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/on.js "Source")

Adds or removes a *listener* to each selected element for the specified event *typenames*. The *typenames* is a string event type, such as `click`, `mouseover`, or `submit`; any [DOM event type](https://developer.mozilla.org/en-US/docs/Web/Events#Standard_events) supported by your browser may be used. The type may be optionally followed by a period (`.`) and a name; the optional name allows multiple callbacks to be registered to receive events of the same type, such as `click.foo` and `click.bar`. To specify multiple typenames, separate typenames with spaces, such as `input change` or `click.foo click.bar`.

When a specified event is dispatched on a selected element, the specified *listener* will be evaluated for the element, being passed the current datum (*d*), the current index (*i*), and the current group (*nodes*), with *this* as the current DOM element (*nodes*[*i*]). Listeners always see the latest datum for their element, but the index is a property of the selection and is fixed when the listener is assigned; to update the index, re-assign the listener. To access the current event within a listener, use [d3.event](#event).

If an event listener was previously registered for the same *typename* on a selected element, the old listener is removed before the new listener is added. To remove a listener, pass null as the *listener*. To remove all listeners for a given name, pass null as the *listener* and `.foo` as the *typename*, where `foo` is the name; to remove all listeners with no name, specify `.` as the *typename*.

An optional *capture* flag may be specified which corresponds to the W3C [useCapture flag](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-registration): “After initiating capture, all events of the specified type will be dispatched to the registered EventListener before being dispatched to any EventTargets beneath them in the tree. Events which are bubbling upward through the tree will not trigger an EventListener designated to use capture.”

If a *listener* is not specified, returns the currently-assigned listener for the specified event *typename* on the first (non-null) selected element, if any. If multiple typenames are specified, the first matching listener is returned.

<a name="selection_dispatch" href="#selection_dispatch">#</a> <i>selection</i>.<b>dispatch</b>(<i>type</i>[, <i>parameters</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/dispatch.js "Source")

Dispatches a [custom event](http://www.w3.org/TR/dom/#interface-customevent) of the specified *type* to each selected element, in order. An optional *parameters* map may be specified to set additional properties of the event. It may contain the following fields:

* [`bubbles`](https://www.w3.org/TR/dom/#dom-event-bubbles) - if true, the event is dispatched to ancestors in reverse tree order.
* [`cancelable`](https://www.w3.org/TR/dom/#dom-event-cancelable) - if true, *event*.preventDefault is allowed.
* [`detail`](https://www.w3.org/TR/dom/#dom-customevent-detail) - any custom data associated with the event.

If *parameters* is a function, it is evaluated for each selected element, in order, being passed the current datum (*d*), the current index (*i*), and the current group (*nodes*), with *this* as the current DOM element (*nodes*[*i*]). It must return the parameters map for the current element.

<a name="event" href="#event">#</a> d3.<b>event</b>

The current [event](https://developer.mozilla.org/en-US/docs/DOM/event), if any. This is set during the invocation of an event listener, and is reset after the listener terminates. Use this to access standard event fields such as [*event*.timeStamp](https://www.w3.org/TR/dom/#dom-event-timestamp) and methods such as [*event*.preventDefault](https://www.w3.org/TR/dom/#dom-event-preventdefault). While you can use the native [*event*.pageX](https://developer.mozilla.org/en/DOM/event.pageX) and [*event*.pageY](https://developer.mozilla.org/en/DOM/event.pageY), it is often more convenient to transform the event position to the local coordinate system of the container that received the event using [d3.mouse](#mouse), [d3.touch](#touch) or [d3.touches](#touches).

If you use Babel, Webpack, or another ES6-to-ES5 bundler, be aware that the value of d3.event changes during an event! An import of d3.event must be a [live binding](http://www.2ality.com/2015/07/es6-module-exports.html), so you may need to configure the bundler to import from D3’s ES6 modules rather than from the generated UMD bundle; not all bundlers observe [jsnext:main](https://github.com/rollup/rollup/wiki/jsnext:main). Also beware of conflicts with the [*window*.event](https://developer.mozilla.org/en-US/docs/Web/API/Window/event) global.

<a name="customEvent" href="#customEvent">#</a> d3.<b>customEvent</b>(<i>event</i>, <i>listener</i>[, <i>that</i>[, <i>arguments</i>]]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/on.js#L98 "Source")

Invokes the specified *listener*, using the specified *that* `this` context and passing the specified *arguments*, if any. During the invocation, [d3.event](#event) is set to the specified *event*; after the listener returns (or throws an error), d3.event is restored to its previous value. In addition, sets *event*.sourceEvent to the prior value of d3.event, allowing custom events to retain a reference to the originating native event. Returns the value returned by the *listener*.

<a name="mouse" href="#mouse">#</a> d3.<b>mouse</b>(<i>container</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/mouse.js "Source")

Returns the *x* and *y* coordinates of the [current event](#event) relative to the specified *container*. The container may be an HTML or SVG container element, such as a [G element](http://www.w3.org/TR/SVG/struct.html#Groups) or an [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement). The coordinates are returned as a two-element array of numbers [*x*, *y*].

<a name="touch" href="#touch">#</a> d3.<b>touch</b>(<i>container</i>[, <i>touches</i>], <i>identifier</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/touch.js "Source")

Returns the *x* and *y* coordinates of the touch with the specified *identifier* associated with the [current event](#event) relative to the specified *container*. The container may be an HTML or SVG container element, such as a [G element](http://www.w3.org/TR/SVG/struct.html#Groups) or an [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement). The coordinates are returned as a two-element array of numbers [*x*, *y*]. If there is no touch with the specified identifier in *touches*, returns null; this can be useful for ignoring touchmove events where the only some touches have moved. If *touches* is not specified, it defaults to the current event’s [changedTouches](http://developer.apple.com/library/safari/documentation/UserExperience/Reference/TouchEventClassReference/TouchEvent/TouchEvent.html#//apple_ref/javascript/instp/TouchEvent/changedTouches) property.

<a name="touches" href="#touches">#</a> d3.<b>touches</b>(<i>container</i>[, <i>touches</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/touches.js "Source")

Returns the *x* and *y* coordinates of the touches associated with the [current event](#event) relative to the specified *container*. The container may be an HTML or SVG container element, such as a [G element](http://www.w3.org/TR/SVG/struct.html#Groups) or an [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement). The coordinates are returned as an array of two-element arrays of numbers \[\[*x1*, *y1*], [*x2*, *y2*], …\]. If *touches* is not specified, it defaults to the current event’s [touches](http://developer.apple.com/library/safari/documentation/UserExperience/Reference/TouchEventClassReference/TouchEvent/TouchEvent.html#//apple_ref/javascript/instp/TouchEvent/touches) property.

<a name="clientPoint" href="#clientPoint">#</a> d3.<b>clientPoint</b>(<i>container</i>, <i>event</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/point.js "Source")

Returns the *x* and *y* coordinates of the specified *event* relative to the specified *container*. (The *event* may also be a [touch](https://www.w3.org/TR/touch-events/#touch-interface).) The container may be an HTML or SVG container element, such as a [G element](http://www.w3.org/TR/SVG/struct.html#Groups) or an [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement). The coordinates are returned as a two-element array of numbers [*x*, *y*].

### Control Flow

For advanced usage, selections provide methods for custom control flow.

<a name="selection_each" href="#selection_each">#</a> <i>selection</i>.<b>each</b>(<i>function</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/each.js "Source")

Invokes the specified *function* for each selected element, in order, being passed the current datum (*d*), the current index (*i*), and the current group (*nodes*), with *this* as the current DOM element (*nodes*[*i*]). This method can be used to invoke arbitrary code for each selected element, and is useful for creating a context to access parent and child data simultaneously, such as:

```js
parent.each(function(p, j) {
  d3.select(this)
    .selectAll(".child")
      .text(function(d, i) { return "child " + d.name + " of " + p.name; });
});
```

See [Sized Donut Multiples](http://bl.ocks.org/mbostock/4c5fad723c87d2fd8273) for an example.

<a name="selection_call" href="#selection_call">#</a> <i>selection</i>.<b>call</b>(<i>function</i>[, <i>arguments…</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/call.js "Source")

Invokes the specified *function* exactly once, passing in this selection along with any optional *arguments*. Returns this selection. This is equivalent to invoking the function by hand but facilitates method chaining. For example, to set several styles in a reusable function:

```js
function name(selection, first, last) {
  selection
      .attr("first-name", first)
      .attr("last-name", last);
}
```

Now say:

```js
d3.selectAll("div").call(name, "John", "Snow");
```

This is roughly equivalent to:

```js
name(d3.selectAll("div"), "John", "Snow");
```

The only difference is that *selection*.call always returns the *selection* and not the return value of the called *function*, `name`.

<a name="selection_empty" href="#selection_empty">#</a> <i>selection</i>.<b>empty</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/empty.js "Source")

Returns true if this selection contains no (non-null) elements.

<a name="selection_nodes" href="#selection_nodes">#</a> <i>selection</i>.<b>nodes</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/nodes.js "Source")

Returns an array of all (non-null) elements in this selection.

<a name="selection_node" href="#selection_node">#</a> <i>selection</i>.<b>node</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/node.js "Source")

Returns the first (non-null) element in this selection. If the selection is empty, returns null.

<a name="selection_size" href="#selection_size">#</a> <i>selection</i>.<b>size</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/size.js "Source")

Returns the total number of elements in this selection.

### Local Variables

D3 locals allow you to define local state independent of data. For instance, when rendering [small multiples](http://bl.ocks.org/mbostock/e1192fe405703d8321a5187350910e08) of time-series data, you might want the same *x*-scale for all charts but distinct *y*-scales to compare the relative performance of each metric. D3 locals are scoped by DOM elements: on set, the value is stored on the given element; on get, the value is retrieved from given element or the nearest ancestor that defines it.

<a name="local" href="#local">#</a> d3.<b>local</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js "Source")

Declares a new local variable. For example:

```js
var foo = d3.local();
```

Like `var`, each local is a distinct symbolic reference; unlike `var`, the value of each local is also scoped by the DOM.

<a name="local_set" href="#local_set">#</a> <i>local</i>.<b>set</b>(<i>node</i>, <i>value</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L18 "Source")

Sets the value of this local on the specified *node* to the *value*, and returns the specified *value*. This is often performed using [*selection*.each](#selection_each):

```js
selection.each(function(d) { foo.set(this, d.value); });
```

If you are just setting a single variable, consider using [*selection*.property](#selection_property):

```js
selection.property(foo, function(d) { return d.value; });
```

<a name="local_get" href="#local_get">#</a> <i>local</i>.<b>get</b>(<i>node</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L13 "Source")

Returns the value of this local on the specified *node*. If the *node* does not define this local, returns the value from the nearest ancestor that defines it. Returns undefined if no ancestor defines this local.

<a name="local_remove" href="#local_remove">#</a> <i>local</i>.<b>remove</b>(<i>node</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L21 "Source")

Deletes this local’s value from the specified *node*. Returns true if the *node* defined this local prior to removal, and false otherwise. If ancestors also define this local, those definitions are unaffected, and thus [*local*.get](#local_get) will still return the inherited value.

<a name="local_toString" href="#local_toString">#</a> <i>local</i>.<b>toString</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L24 "Source")

Returns the automatically-generated identifier for this local. This is the name of the property that is used to store the local’s value on elements, and thus you can also set or get the local’s value using *element*[*local*] or by using [*selection*.property](#selection_property).

### Namespaces

XML namespaces are fun! Right? Fortunately you can mostly ignore them.

<a name="namespace" href="#namespace">#</a> d3.<b>namespace</b>(<i>name</i>) [<>源码](https://github.com/d3/d3-selection/blob/master/src/namespace.js "Source")

Qualifies the specified *name*, which may or may not have a namespace prefix. If the name contains a colon (`:`), the substring before the colon is interpreted as the namespace prefix, which must be registered in [d3.namespaces](#namespaces). Returns an object `space` and `local` attributes describing the full namespace URL and the local name. For example:

```js
d3.namespace("svg:text"); // {space: "http://www.w3.org/2000/svg", local: "text"}
```

If the name does not contain a colon, this function merely returns the input name.

<a name="namespaces" href="#namespaces">#</a> d3.<b>namespaces</b> [<源码>](https://github.com/d3/d3-selection/blob/master/src/namespaces.js "Source")

The map of registered namespace prefixes. The initial value is:

```js
{
  svg: "http://www.w3.org/2000/svg",
  xhtml: "http://www.w3.org/1999/xhtml",
  xlink: "http://www.w3.org/1999/xlink",
  xml: "http://www.w3.org/XML/1998/namespace",
  xmlns: "http://www.w3.org/2000/xmlns/"
}
```

Additional prefixes may be assigned as needed to create elements or attributes in other namespaces.
