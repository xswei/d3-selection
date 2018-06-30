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

与 [*selection*.selectAll](#selection_selectAll) 不同, *selection*.select 不会影响分组: 它会保存现有的分组结构以及索引并且将关联数据(如果有的话)传递给选中的子节点。分组在 [data join](#joining-data) 中扮演着很重要的角色。参考 [Nested Selections](http://bost.ocks.org/mike/nest/) 和 [How Selections Work](http://bost.ocks.org/mike/selection/) 获取更多信息.

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

要注意的是 `:nth-child` 伪类基于 1 的索引而不是基于 0 开始。此外上述过滤函数与 `:nth-child` 的含义完全不同，一个是基于索引而另一个是基于 DOM 中兄弟元素的数量。

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

重新将元素插入到文档中以使得文档中每个分组的次序与选择集的次序匹配。如果数据已经有序的话，这个方法与 [*selection*.sort](#selection_sort) 等效，但是要更快。

<a name="selection_raise" href="#selection_raise">#</a> <i>selection</i>.<b>raise</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/raise.js "Source")

按序重新插入每个选中的元素，每次插入的元素都作为其父元素的最后一个子元素。等价于:

```js
selection.each(function() {
  this.parentNode.appendChild(this);
});
```

<a name="selection_lower" href="#selection_lower">#</a> <i>selection</i>.<b>lower</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/lower.js "Source")

按序重新插入每个选中的元素，每次插入的元素都作为其父元素的第一个子元素。等价于:

```js
selection.each(function() {
  this.parentNode.insertBefore(this, this.parentNode.firstChild);
});
```

<a name="create" href="#create">#</a> d3.<b>create</b>(<i>name</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/create.js "Source")

根据指定的元素 *name* 返回一个与当前文档分离的单元素选择集，这个选择集中包含的元素名为指定的 *name*。

<a name="creator" href="#creator">#</a> d3.<b>creator</b>(<i>name</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/creator.js "Source")

根据指定元素的 *name* 返回一个创建该 *name* 元素的函数，假定 `this` 是其父元素。这个方法内部通过 [*selection*.append](#selection_append) 和 [*selection*.insert](#selection_insert) 来创建新元素。例如：

```js
selection.append("div");
```

等价于:

```js
selection.append(d3.creator("div"));
```

参考 [namespace](#namespace) 获取支持的命名空间前缀，例如 SVG 元素.

### Joining Data

关于 D3’s 数据绑定的介绍可以参考 [Thinking With Joins](http://bost.ocks.org/mike/join/). 也可以参考例子  [General Update Pattern](http://bl.ocks.org/mbostock/3808218).

<a name="selection_data" href="#selection_data">#</a> <i>selection</i>.<b>data</b>([<i>data</i>[, <i>key</i>]]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/data.js "Source")

将指定数组的数据 *data* 与已经选中的元素进行绑定并返回一个新的选择集，返回的新的选择集使用 *update* 表示: 此时数据已经成功的与元素绑定。并且定义了 [enter](#selection_enter) 和 [exit](#selection_exit) 方法用来返回需要加入元素和移除元素的选择集。*data* 可以是任意数据类型的数组(*e.g.*, 数值或对象), 可以是一个返回数组的方法(比如为每个分组继续绑定数组时). 当数据分配给元素时，会被存储在元素的 `__data__` 属性上, 因此可以在重新选中元素时继续使用与元素对应的数据。

*data* 会被指定给选择集中的 **each group(每个分组)**。如果选择集中包含多个分组(比如 [d3.selectAll](#selectAll) 后跟随 [*selection*.selectAll](#selection_selectAll))，则 *data* 应该应该被指定为一个函数。这个函数会为每个分组进行调用, 并依次传递当前分组的数据(*d*, 可能没有定义), 分组的索引 (*i*), 以及选择集的父节点 (*nodes*), 函数中 *this* 指向当前分组的父元素。例如根据如下数值矩阵创建一个 `HTML` 表格:

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

在这个例子中 *data* 函数时一个恒等函数: 对于每一行，返回矩阵中对应的行数据.

如果没有指定 *key* 函数，则第一个数据 *data* 被分配给第一个被选中的元素，第二个数据 *data* 被分配给第二个选中的元素, 以此类推. *key* 函数通过计算每个数据与元素的字符串标识来控制哪一个数据与哪一个元素绑定, 替代默认的按索引绑定的方式。*key* 函数为每一个选中的元素进行调用并传递当前元素绑定的数据 *d*, 当前的索引 *i*, 以及当前的分组 *nodes*, 函数内部 *this* 指向当前的 `DOM` 节点(*nodes*[*i*]); 返回的字符串为元素的 *key* 标识. *key* 函数也会为每个新的数据元素 *data* 进行调用, 并传递当前的数据 *d*, 当前索引 *i*, 以及分组的新数据 *data*, 函数内部 *this* 指向当前分组的父级 `DOM` 元素; 返回的字符串为当前数据的 *key* 标识。每个数据会被分配给与之相匹配 *key* 标识的元素。如果多个元素有相同的 *key* 标识则重复的元素会被放入 `exit` 选择集中; 如果多个数据有相同的 *key*, 则多余的数据会被放入到 `enter` 选择集中.

例如给定一下文档:

```html
<div id="Ford"></div>
<div id="Jarrah"></div>
<div id="Kwon"></div>
<div id="Locke"></div>
<div id="Reyes"></div>
<div id="Shephard"></div>
```

你可以使用如下方法来绑定数据：


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

这个例子中的 *key* 函数使用了数据 *d*, 如果不存在的话返回元素的 `id` 属性。因为这些元素没有被绑定数据，因此当每个元素被调用 *key* 函数时 *d* 为 `null` 而为新数据调用时非空.

*update* 和 *enter* 选择集根据数据的顺序返回, *exit* 选择集保留数据绑定之前的顺序。如果 *key* 函数被指定, 则选择集中的元素顺序与文档中的顺序可能不匹配, 此时需要使用 [*selection*.order](#order) 或 [*selection*.sort](#sort)。更多关于 *key* 函数如何影响数据绑定的资料，可以参考 [A Bar Chart, Part 2](http://bost.ocks.org/mike/bar/2/) 和 [Object Constancy](http://bost.ocks.org/mike/constancy/).

尽管数据绑定可以用来简单的创建 (*enter*) 一组与数据对应的元素，更一般的说, 数据绑定的设计是为了让你根据需要创建, 销毁或者更新元素，以使得 `DOM` 元素与新数据集对应. 数据绑定帮助你更高效的完成这些操作, 你只需要在每个元素状态(新加, 更新以及移除) 上执行最少且必要的操作即可, 此外还允许在这些元素状态上执行一些过渡效果。这是一个 [General Update Pattern(通用更新模式)](http://bl.ocks.org/mbostock/3808218) 的简单示例:

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

分解为以下几步:

1. 任何现存的圆( `SVG` 的后代元素) 被 [selected](#selection_selectAll).
2. 这些圆被 [绑定新 `data`](#selection_data), 返回与之前对应的圆: *update* 选择集.
3. 这些更新的圆使用蓝色填充.
4. 多余的圆(也就是已经存在的圆没有匹配到新数据中的任何一个元素)被移除: *exit* 选择集.
5. 新的圆(没有与新数据对应的圆元素)被 [appended](#selection_append): *enter* selection.
6. 新添加的圆使用绿色填充.
7. 新的圆与已经存在的圆进行 [union](#selection_merge).
8. 所有的圆(已有的和被新加的)被设置为黑色描边.
9. 使用 `circle` 变量存储合并之后的选择集.

正如上文所述, “匹配”逻辑使由 *key* 函数决定并传递给 *selection*.data 的; 如果没有指定 *key* 函数则元素和数据会按照索引绑定。

如果没有指定 *data* 则这个方法会返回当前被选中元素绑定的数据数组.

这个方法不能用于清除数据绑定，可以使用 [*selection*.datum](#selection_datum) 替代。

<a name="selection_enter" href="#selection_enter">#</a> <i>selection</i>.<b>enter</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/enter.js "Source")

返回 `enter` 选择集: 没有对应 `DOM` 节点的数据的占位节点. (对于不是通过 [*selection*.data](#selection_data) 返回的选择集 `enter` 选择集为空)

`enter` 选择集通常在数据比节点多时用来创建缺失的节点。比如根据以下数据创建 `DIV` 元素:

```js
var div = d3.select("body")
  .selectAll("div")
  .data([4, 8, 15, 16, 23, 42])
  .enter().append("div")
    .text(function(d) { return d; });
```

如果 `body` 初始为空，则上述代码会创建 6 个新的 `DIV` 元素并依次添加到 `body` 中，并且将其文本内容设置为对应的数值:

```html
<div>4</div>
<div>8</div>
<div>15</div>
<div>16</div>
<div>23</div>
<div>42</div>
```

从概念上来讲，`enter` 选择集的占位符是一个指向父元素的指针(上述例子中为 `body`)。`enter` 选择集通常仅仅用来添加元素，并且在添加完元素之后与 `update` 选择集进行 [merged](#selection_merge), 这样的话数据的修改可以同时应用于 `enter` 的元素和 `update` 的元素。

<a name="selection_exit" href="#selection_exit">#</a> <i>selection</i>.<b>exit</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/exit.js "Source")

返回 `exit` 选择集: 没有对应数据的已经存在的 `DOM` 节点。(对于不是通过 [*selection*.data](#selection_data) 返回的选择集 `exit` 选择集为空)

`exit` 选择集通常用来移除多余的元素。例如使用新的数据更新 `DIV` 元素:

```js
div = div.data([1, 2, 4, 8, 16, 32], function(d) { return d; });
```

因为指定了 *key* 函数(恒等函数), 并且新的数据包含数值 [4, 8, 16] 能匹配到已经存在的元素, `update` 选择集包含三个 `DIV` 元素。保留已经存在的并且能匹配数据的元素，然后通过 `enter` 选择集为 [1, 2, 32] 添加新的元素:

```js
div.enter().append("div").text(function(d) { return d; });
```

同样的, 移除与 [15, 23, 42] 绑定的元素:

```js
div.exit().remove();
```

此时文档结构如下:

```html
<div>1</div>
<div>2</div>
<div>4</div>
<div>8</div>
<div>16</div>
<div>32</div>
```

`DOM` 元素的顺序与数据的顺序一致，因为旧数据的顺序和新数据的顺序是一致的( `enter` 选择集长度与数据长度一致，只不过中间会有 `empty` 表示不需要插入元素, 因此能保证次序)。如果新元素的次序发生了改变则可以使用 [*selection*.order](#selection_order) 来重新排列 `DOM` 元素的次序。参考 [General Update Pattern](http://bl.ocks.org/mbostock/3808218) 来查看数据绑定的例子。

<a name="selection_datum" href="#selection_datum">#</a> <i>selection</i>.<b>datum</b>([<i>value</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/datum.js "Source")

获取或设置每个选中元素上绑定的数据。与 [*selection*.data](#selection_data) 不同, 这个方法不会进行数据链接计算并且不影响索引, 不影响 `enter` 和 `exit` 选择集.

如果指定了 *value*, 则将当前所有选中的元素绑定的数据设置为指定的值。如果 *value* 为常量则所有的元素的绑定的数据会被设置为相同的值，如果 *value* 为函数，则会为每个选中的元素进行调用，并传递当前元素绑定的数据 *d*, 当前的索引 *i*, 当前分组 *nodes*, 函数内部 *this* 指向当前 `DOM` 元素(*nodes*[*i*]). 这个函数用来设置设置每个元素绑定的新数据. 如果为 `null` 则表示移除当前元素绑定的数据。

如果没有指定 *value* 则返回当前所有选中的元素中第一个非空元素所绑定的数据。当已知选择集中只包含一个元素时候通常很有用。

这个方法在访问 `HTML5` 的 [custom data attributes](http://www.w3.org/TR/html5/dom.html#custom-data-attribute) 时很有用. 比如, 给定如下元素:

```html
<ul id="list">
  <li data-username="shawnbot">Shawn Allen</li>
  <li data-username="mbostock">Mike Bostock</li>
</ul>
```

你可以通过将每个元素绑定的数据设置为其 [dataset](http://www.w3.org/TR/html5/dom.html#dom-dataset) 属性:

```js
selection.datum(function() { return this.dataset; })
```

### Handling Events

为了交互，选择集允许监听和分派事件。

<a name="selection_on" href="#selection_on">#</a> <i>selection</i>.<b>on</b>(<i>typenames</i>[, <i>listener</i>[, <i>capture</i>]]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/on.js "Source")

为每个选中的元素添加或者移除一个指定*typenames* 事件的 *listener*. *typenames* 是一个字符串表示的事件类型, 比如 `click`, `mouseover`, 或者 `submit`; 浏览器支持的 [DOM event type](https://developer.mozilla.org/en-US/docs/Web/Events#Standard_events) 都可以使用。类型后面可以跟一个由( `.` )分割的可选的 `name`；可选的名称允许在同一个事件类型上注册多个回调, 比如 `click.foo` and `click.bar`. 同时指定多个 `typenames` 可以使用空格隔开，比如 `input change` 或 `click.foo click.bar`.

当指定的事件被分配给已选中的元素时，该元素对应的 *listener* 将会被调用, 并传递当前绑定的数据 *d*, 当前的索引 *i*, 当前的分组 (*nodes*), 回调中 *this* 指向当前的 `DOM` 元素(*nodes*[*i*])。事件监听器只能获取到元素上最新绑定的数据，而索引是选择集的一个属性并且在分配监听器的时候已经被固定，如果需要更新索引则需要重新分配事件监听器。在监听器内使用 [d3.event](#event) 访问当前的事件。

如果在一个已选中的元素上已经注册了相同 *typename* 的监听器, 则旧的会被移除而新的监听器会被添加。如果要移除一个事件监听器，可以将 *listener* 设置为 `null`。移除所有的指定 *name* 的监听器可以将 *typename* 设置为 `.foo`, 同时将 *listener* 设置为 `null`, 其中 `foo` 为名称。移除所有没有名称的监听器，则将 *typename* 设置为 (`.`)。

可选的 *capture* 标志位与 `W3C` 的 [useCapture flag](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-registration) 对应: “启动捕获后, 所有的指定类型的事件将会被分发给指定的 `EventListener` 然后被分派到树中任何 `EventTargets`. 向上冒泡的事件不会触发指定使用捕获的 `EventListener`.”

如果没有指定 *listener*，则返回当前第一个非空元素对应的 *typename* 事件监听器. 如果指定了多个 `typename` 则返回第一个匹配的事件监听器.

<a name="selection_dispatch" href="#selection_dispatch">#</a> <i>selection</i>.<b>dispatch</b>(<i>type</i>[, <i>parameters</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/dispatch.js "Source")

为每个选中的元素根据指定的 *type* 按序派发一个 [custom event](http://www.w3.org/TR/dom/#interface-customevent). 可以指定一个可选的 *parameters* 映射来设置事件的附加属性。包含以下几个字段:

* [`bubbles`](https://www.w3.org/TR/dom/#dom-event-bubbles) - 如果为 `true`, 这个事件会冒泡给祖先节点.
* [`cancelable`](https://www.w3.org/TR/dom/#dom-event-cancelable) - 如果为 `true`, 则 *event*.preventDefault 会被允许.
* [`detail`](https://www.w3.org/TR/dom/#dom-customevent-detail) - 任何关联事件的其他自定义数据.

如果 *parameters* 是一个函数，则会为每个选中的元素进行依次调用, 并传递当前的数据 *d*, 当前的索引 *i* 以及当前分组 *nodes*，函数内部 *this* 指向当前 `DOM` 元素。必须为当前元素返回一个 *parameters* 映射.

<a name="event" href="#event">#</a> d3.<b>event</b>

当前 [event](https://developer.mozilla.org/en-US/docs/DOM/event) (如果存在的话). 这个值在调用事件监听器时设置的，并且在监听器执行结束之后重置。使用这个属性可以访问标准的事件字段，比如 [*event*.timeStamp](https://www.w3.org/TR/dom/#dom-event-timestamp) 以及方法比如 [*event*.preventDefault](https://www.w3.org/TR/dom/#dom-event-preventdefault). 你可以使用原生的 [*event*.pageX](https://developer.mozilla.org/en/DOM/event.pageX) 和 [*event*.pageY](https://developer.mozilla.org/en/DOM/event.pageY) 以方便的将事件位置转换为使用 [d3.mouse](#mouse), [d3.touch](#touch) 或 [d3.touches](#touches) 接收事件的容器的本地坐标。

如果你使用 `Babel`, `Webpack` 或者其他的 `ES6` 转 `ES5` 的打包工具，要注意 d3.event 的值在事件中的变化！导入的 d3.event 必须是 [live binding(动态绑定)](http://www.2ality.com/2015/07/es6-module-exports.html) 的，因此你需要将打包配置设置为引入 `D3` 的 `ES6` 模块而不是生成的 `UMD`；并不是所有的打包工具都识别 [jsnext:main](https://github.com/rollup/rollup/wiki/jsnext:main)。也要注意与 [*window*.event](https://developer.mozilla.org/en-US/docs/Web/API/Window/event) 的冲突。

<a name="customEvent" href="#customEvent">#</a> d3.<b>customEvent</b>(<i>event</i>, <i>listener</i>[, <i>that</i>[, <i>arguments</i>]]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/on.js#L98 "Source")

调用指定的 *listener*, 使用指定的 *that* 作为 `this` 上下文并传递指定的 *arguments*(如果有的话). 在调用期间，[d3.event](#event) 被设置为指定的 *event*; 当事件监听器返回(或抛出错), d3.event 被重置为之前的值。此外，设置 *event*.sourceEvent 为 d3.event 的预设值，允许自定义事件对原生事件的引用。返回由 *listener* 返回的值。

<a name="mouse" href="#mouse">#</a> d3.<b>mouse</b>(<i>container</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/mouse.js "Source")

返回 [current event](#event) 相对于指定 *container* 的 *x* 和 *y* 坐标。`container` 可以是一个 `HTML` 或 `SVG` 容器元素，比如 [G element](http://www.w3.org/TR/SVG/struct.html#Groups) 或者 [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement)。坐标以二元数组的形式返回: [*x*, *y*].

<a name="touch" href="#touch">#</a> d3.<b>touch</b>(<i>container</i>[, <i>touches</i>], <i>identifier</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/touch.js "Source")

返回与 [current event](#event) 相关的指定标识符的触摸事件相对于指定 *container* 的 *x* 和 *y* 坐标。`container` 可以是一个 `HTML` 或 `SVG` 容器元素，比如 [G element](http://www.w3.org/TR/SVG/struct.html#Groups) 或者 [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement)。坐标以二元数组的形式返回: [*x*, *y*]. 如果当前 *touches* 中没有与指定标识符相关联的事件则返回 `null`。当仅仅有部分触摸移动的时候忽略 `touchmove` 会很有用。如果没有指定 *touches* 则默认为当前事件的 [changedTouches](http://developer.apple.com/library/safari/documentation/UserExperience/Reference/TouchEventClassReference/TouchEvent/TouchEvent.html#//apple_ref/javascript/instp/TouchEvent/changedTouches) 属性

<a name="touches" href="#touches">#</a> d3.<b>touches</b>(<i>container</i>[, <i>touches</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/touches.js "Source")

返回 [current event(当前触摸事件)](#event) 相对于指定 *container* 的 *x* 和 *y* 坐标。`container` 可以是一个 `HTML` 或 `SVG` 容器元素，比如 [G element](http://www.w3.org/TR/SVG/struct.html#Groups) 或者 [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement)。返回的坐标以二维数组的形式返回: \[\[*x1*, *y1*], [*x2*, *y2*], …\]. 如果 *touches* 没有被指定则默认为当前事件的 [touches](http://developer.apple.com/library/safari/documentation/UserExperience/Reference/TouchEventClassReference/TouchEvent/TouchEvent.html#//apple_ref/javascript/instp/TouchEvent/touches) 属性。

<a name="clientPoint" href="#clientPoint">#</a> d3.<b>clientPoint</b>(<i>container</i>, <i>event</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/point.js "Source")

返回指定 *event* 相对于指定 *container* 的 *x* 和 *y* 坐标。`container` 可以是一个 `HTML` 或 `SVG` 容器元素，比如 [G element](http://www.w3.org/TR/SVG/struct.html#Groups) 或者 [SVG element](http://www.w3.org/TR/SVG/struct.html#SVGElement)。坐标以二元数组的形式返回: [*x*, *y*].

### Control Flow

更高阶的应用就是选择集提供了定制化的控制流。

<a name="selection_each" href="#selection_each">#</a> <i>selection</i>.<b>each</b>(<i>function</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/each.js "Source")

为每个选中的元素依次调用指定的 *function*，并传递当前元素绑定的数据 *d*，当前索引 *i* 以及当前分组 *nodes*. 函数内部 *this* 指向当前 DOM 元素(*node*[*i*]). 这个方法可以为选中的每个元素调用任意代码，在同一个作用域中同时访问父节点和子节点数据时很有用，比如:

```js
parent.each(function(p, j) {
  d3.select(this)
    .selectAll(".child")
      .text(function(d, i) { return "child " + d.name + " of " + p.name; });
});
```

参考 [Sized Donut Multiples](http://bl.ocks.org/mbostock/4c5fad723c87d2fd8273).

<a name="selection_call" href="#selection_call">#</a> <i>selection</i>.<b>call</b>(<i>function</i>[, <i>arguments…</i>]) [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/call.js "Source")

调用一次指定的 *function*，并为将当前选择集作为第一个参数，此外还可以使用可选的其他参数。返回选择集，等价于手动调用函数，但是这种方法支持链式调用。例如有一个设置样式的函数:

```js
function name(selection, first, last) {
  selection
      .attr("first-name", first)
      .attr("last-name", last);
}
```

可以写成如下形式:

```js
d3.selectAll("div").call(name, "John", "Snow");
```

等价于：

```js
name(d3.selectAll("div"), "John", "Snow");
```

不同的是 *selection*.call 总是返回当前选择集，而不是 *function* 的返回值。

<a name="selection_empty" href="#selection_empty">#</a> <i>selection</i>.<b>empty</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/empty.js "Source")

当且仅当当前选择集中没有任何元素时返回 true.

<a name="selection_nodes" href="#selection_nodes">#</a> <i>selection</i>.<b>nodes</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/nodes.js "Source")

返回选择集中被选中元素的元素数组。

<a name="selection_node" href="#selection_node">#</a> <i>selection</i>.<b>node</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/node.js "Source")

返回选择集中第一个非空元素。如果选择集为空则返回 null。

<a name="selection_size" href="#selection_size">#</a> <i>selection</i>.<b>size</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/selection/size.js "Source")

返回选择集中包含的元素个数。

### Local Variables

D3 局部变量允许定义数据的局部状态。例如在渲染 [small multiples](http://bl.ocks.org/mbostock/e1192fe405703d8321a5187350910e08) 时序数据时，可能 *x* 比例尺可以共用一个，但是 *y* 比例尺可能会有不同的度量标准。D3 局部变量被限制在 DOM 元素上: 在设置时，该值存储在元素上，在获取时则从给定元素或其祖先节点查找定义的该局部变量的值。

<a name="local" href="#local">#</a> d3.<b>local</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js "Source")

声明一个新的局部变量。例如：

```js
var foo = d3.local();
```

与 `var` 类似都是一个声明引用。但是与 `var` 不同的是局部变量被定义在 DOM 元素。

<a name="local_set" href="#local_set">#</a> <i>local</i>.<b>set</b>(<i>node</i>, <i>value</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L18 "Source")

将指定的 *value* 设置为指定 *node* 的局部变量，并返回指定的 *value*。这个方法经常和 [*selection*.each](#selection_each) 结合使用:

```js
selection.each(function(d) { foo.set(this, d.value); });
```

如果设置的是单一变量，则考虑使用 [*selection*.property](#selection_property):

```js
selection.property(foo, function(d) { return d.value; });
```

<a name="local_get" href="#local_get">#</a> <i>local</i>.<b>get</b>(<i>node</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L13 "Source")

返回指定 *node* 的局部变量。如果 *node* 没有被定义局部变量，则返回最近的定义了局部变量的祖先节点的局部变量。如果都没有定义的话返回 `undefined`.

<a name="local_remove" href="#local_remove">#</a> <i>local</i>.<b>remove</b>(<i>node</i>) [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L21 "Source")

移除指定 *node* 的局部变量，当且仅当指定的节点定义了局部变量时返回 `true`，否则返回 `false`。如果祖先节点也定义了这个局部变量，则不会受影响，因此使用 [*local*.get](#local_get) 仍然会返回继承自祖先节点的值。

<a name="local_toString" href="#local_toString">#</a> <i>local</i>.<b>toString</b>() [<源码>](https://github.com/d3/d3-selection/blob/master/src/local.js#L24 "Source")

返回自动生成的局部变量的标识符。这个标识符是存储在元素上的局部变量的属性名，因此还可以使用 *element*[*local*] 或使用 [*selection*.property](#selection_property) 来设置或获取局部变量。

### Namespaces

XML 的命名空间很有意思，不是吗？幸运的是你大多数情况下都可以忽略它们。

<a name="namespace" href="#namespace">#</a> d3.<b>namespace</b>(<i>name</i>) [<>源码](https://github.com/d3/d3-selection/blob/master/src/namespace.js "Source")

限定指定的命名，可能不包含命名空间前缀。如果name包含了冒号(:)，则冒号之前的子串被作为命名空间前缀注册到[d3.namespaces](#namespaces). 返回一个包含 `space` 和 `local` 属性的对象用以描述完整的命名空间 `URL` 和 `local` 名称，例如:

```js
d3.namespace("svg:text"); // {space: "http://www.w3.org/2000/svg", local: "text"}
```

如果 `name` 不包含冒号，则直接返回输入的 `name`.

<a name="namespaces" href="#namespaces">#</a> d3.<b>namespaces</b> [<源码>](https://github.com/d3/d3-selection/blob/master/src/namespaces.js "Source")

注册的命名空间前缀。初始值为：

```js
{
  svg: "http://www.w3.org/2000/svg",
  xhtml: "http://www.w3.org/1999/xhtml",
  xlink: "http://www.w3.org/1999/xlink",
  xml: "http://www.w3.org/XML/1998/namespace",
  xmlns: "http://www.w3.org/2000/xmlns/"
}
```

在其他命名空间中创建元素或者属性时候可以使用附加的命名空间前缀.