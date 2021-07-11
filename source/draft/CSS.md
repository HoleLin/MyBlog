# CSS

### 1. 层叠

* Stylesheets **cascade（样式表层叠）** — 简单的说，css规则的顺序很重要；当应用两条同级别的规则到一个元素的时候，写在后面的就是实际使用的规则。

* 优先级
  * 浏览器是根据优先级来决定当多个规则有不同选择器对应相同的元素的时候需要使用哪个规则。它基本上是一个衡量选择器具体选择哪些区域的尺度：
    * 一个元素选择器不是很具体 — 会选择页面上该类型的所有元素 — 所以它的优先级就会低一些。
    * 一个类选择器稍微具体点 — 它会选择该页面中有特定 `class` 属性值的元素 — 所以它的优先级就要高一点。
    * **优先级: 类选择器>元素选择器**
  
* 继承
  
* 继承也需要在上下文中去理解 —— 一些设置在父元素上的css属性是可以被子元素继承的，有些则不能。
  
* 控制继承
  * CSS 为控制继承提供了四个特殊的通用属性值。每个css属性都接收这些值。
    * **inherit**: 设置该属性会使子元素属性和父元素相同。实际上，就是 "开启继承".
    * **initial**: 设置属性值和浏览器默认样式相同。如果浏览器默认样式中未设置且该属性是自然继承的，那么会设置为 `inherit` 。
    * **unset**: 将属性重置为自然值，也就是如果属性是自然继承那么就是 `inherit`，否则和 `initial`一样
  * 重设所有属性值
    * CSS 的 shorthand 属性 `all` 可以用于同时将这些继承值中的一个应用于（几乎）所有属性。它的值可以是其中任意一个(`inherit`, `initial`, `unset`, or `revert`)。这是一种撤销对样式所做更改的简便方法，以便回到之前已知的起点。

* 理解层叠

  * 有三个因素需要考虑，根据重要性排序如下，前面的更重要:

    * **重要程度**

    * **优先级**

      * 一个元素选择器比类选择器的优先级更低会被其覆盖。本质上，不同类型的选择器有不同的分数值，把这些分数相加就得到特定选择器的权重，然后就可以进行匹配。

      * 一个选择器的优先级可以说是由四个部分相加 (分量)，可以认为是个十百千 — 四位数的四个位数：

        * **千位**： 如果声明在 `style` 的属性（内联样式）则该位得一分。这样的声明没有选择器，所以它得分总是1000。

        * **百位**： 选择器中包含ID选择器则该位得一分。

        * **十位**： 选择器中包含类选择器、属性选择器或者伪类则该位得一分。

        * **个位**：选择器中包含元素、伪元素选择器则该位得一分。

        * **通用选择器 (`*`)，组合符 (`+`, `>`, `~`, ' ')，和否定伪类 (`:not`) 不会影响优先级。**

          | 选择器                                  | 千位 | 百位 | 十位 | 个位 | 优先级 |
          | --------------------------------------- | ---- | ---- | ---- | ---- | ------ |
          | h1                                      | 0    | 0    | 0    | 1    | 0001   |
          | h1 + p::first-letter                    | 0    | 0    | 0    | 3    | 0003   |
          | li > a[href*="en-US"] > .inline-warning | 0    | 0    | 2    | 2    | 0022   |
          | #identifier                             | 0    | 1    | 0    | 0    | 0100   |
          | 内联样式                                | 1    | 0    | 0    | 0    | 1000   |
    
       * **!important**: 有一个特殊的 CSS 可以用来覆盖所有上面所有优先级计算，不过需要很小心的使用 — `!important`。用于修改特定属性的值， 能够覆盖普通规则的层叠。

          * 覆盖 `!important` 唯一的办法就是另一个 `!important` 具有 相同*优先级* 而且顺序靠后，或者更高优先级。
    
    * **资源顺序**: 如果你有超过一条规则，而且都是相同的权重，那么最后面的规则会应用。可以理解为后面的规则覆盖前面的规则，直到最后一个开始设置样式。
    
  * CSS位置的影响

    * 相互冲突的声明将按以下顺序适用，后一种声明将覆盖前一种声明：
      * 用户代理样式表中的声明(例如，浏览器的默认样式，在没有设置其他样式时使用)。
      * 用户样式表中的常规声明(由用户设置的自定义样式)。
      * 作者样式表中的常规声明(这些是我们web开发人员设置的样式)。
      * 作者样式表中的`!important`声明
      * 用户样式表中的`!important` 声明

### 2.选择器

  * **类型** :**类型选择器**有时也叫做“标签名选择器*”*或者是”元素选择器“，因为它在文档中选择了一个HTML标签/元素的缘故。

    ```css
    h1 { }
    ```

* **全局选择器**: 全局选择器，是由一个星号（`*`）代指的，它选中了文档中的所有内容（或者是父元素中的所有内容，比如，它紧随在其他元素以及邻代运算符之后的时候）;

  ```css
  * {
      margin: 0;
  }
  article :first-child {
  
  }
  article *:first-child {
  
  } 
  ```

  * **类**:类选择器以一个句点（`.`）开头，会选择文档中应用了这个类的所有物件.

    ```css
    .box { }
    ```

    * 指定特定元素的类 

      * 建立一个指向应用一个类的特定元素
      * 通过使用附加了类的欲选元素的选择器做到这点，其间没有空格。

      ```css
      span.highlight {
          background-color: yellow;
      }
      
      h1.highlight {
          background-color: pink;
      }
      ```

    * 多个类被应用的时候指向一个元素

      ```css
      .notebox {
        border: 4px solid #666;
        padding: .5em;
      }
      
      .notebox.warning {
        border-color: orange;
        font-weight: bold;
      }
      
      .notebox.danger {
        border-color: red;
        font-weight: bold;
      }
      <div class="notebox">
          This is an informational note.
      </div>
      
      <div class="notebox warning">
          This note shows a warning.
      </div>
      
      <div class="notebox danger">
          This note shows danger!
      </div>
      
      <div class="danger">
          This won't get styled — it also needs to have the notebox class
      </div>
      ```

  * **ID选择器**: ID选择器开头为`#`而非句点，不过基本上和类选择器是同种用法

    ```css
    #one {
        background-color: yellow;
    }
    
    h1#heading {
        color: rebeccapurple;
    }
        
    ```

  * **标签属性选择器**

    * 这组选择器根据一个元素上的某个标签的属性的存在以选择元素的不同方式：

    ```css
    a[title] { }
    ```

    * 或者根据一个有特定值的标签属性是否存在来选择：

    ```css
    a[href="https://example.com"] { }
    ```

    * **存否和值选择器**: 这些选择器允许基于一个元素自身是否存在（例如`href`）或者基于各式不同的按属性值的匹配，来选取元素。

    | 选择器             | 示例                          | 描述                                                         |
    | ------------------ | ----------------------------- | ------------------------------------------------------------ |
    | [*attr*]           | a[title]                      | 匹配带有一个名为*attr*的属性的元素——方括号里的值。           |
    | [*attr*=*value*]   | a[href="https://example.com"] | 匹配带有一个名为*attr*的属性的元素，其值正为*value*——引号中的字符串。 |
    | [*attr*~=*value*]  | `p[class~="special"]`         | 匹配带有一个名为*attr*的属性的元素 ，其值正为*value*，或者匹配带有一个*attr*属性的元素，其值有一个或者更多，至少有一个和*value*匹配。注意，在一列中的好几个值，是用空格隔开的。 |
    | [*attr*\|=*value*] | div[lang\|="zh"]              | 匹配带有一个名为*attr*的属性的元素，其值可正为*value*，或者开始为*value*，后面紧随着一个连字符。 |

    * **字符串匹配选择器**:这些选择器让更高级的属性的值的子字符串的匹配变得可行。

    | 选择器            | 示例              | 描述                                                         |
    | ----------------- | ----------------- | ------------------------------------------------------------ |
    | [*attr*^=*value*] | li[class^="box-"] | 匹配带有一个名为*attr*的属性的元素，其值开头为*value*子字符串。 |
    | [*attr*$=*value*] | li[class$="-box"] | 匹配带有一个名为*attr*的属性的元素，其值结尾为*value*子字符串 |
    | [*attr**=*value*] | li[class*="box"]  | 匹配带有一个名为*attr*的属性的元素，其值的字符串中的任何地方，至少出现了一次*value*子字符串。 |

    * 大小写敏感

      * 如果你想在大小写不敏感的情况下，匹配属性值的话，你可以在闭合括号之前，使用`i`值。这个标记告诉浏览器，要以大小写不敏感的方式匹配ASCII字符。没有了这个标记的话，值会按照文档语言对大小写的处理方式，进行匹配——HTML中是大小写敏感的。

      ```css
      li[class^="a"] {
          background-color: yellow;
      }
      
      li[class^="a" i] {
          color: red;
      }
      ```

      * 此外还有一个更加新的`s`值，它会强制在上下文的匹配正常为大小写不敏感的时候，强行要求匹配时大小写敏感。不过，在浏览器中它不太受支持，而且在上下文为HTML时也没啥用

  * **伪类和伪元素**

    * 这组选择器包含了伪类，用来样式化一个元素的特定状态。例如`:hover`伪类会在鼠标指针悬浮到一个元素上的时候选择这个元素：

    ```css
    a:hover { }
    ```

    * 它还可以包含了伪元素，选择一个元素的某个部分而不是元素自己。例如，`::first-line`是会选择一个元素（下面的情况中是`<p>`）中的第一行，类似`<span>`包在了第一个被格式化的行外面，然后选择这个`<span>`。

    ```css
    p::first-line { }
    ```

    * 伪类是选择器的一种，它用于选择处于特定状态的元素，比如当它们是这一类型的第一个元素时，或者是当鼠标指针悬浮在元素上面的时候。

    ```css
    伪类就是开头为冒号的关键字：
    :pseudo-class-name
    
    :first-child 
    :last-child 代表父元素的最后一个子元素。
    :only-child  匹配没有任何兄弟元素的元素.等效的选择器还可以写成 :first-child:last-child或者:nth-child(1):nth-last-child(1),当然,前者的权重会低一点
    :invalid 表示任意内容未通过验证的 <input> 或其他 <form> 元素 .
    ```

    * 用户行为伪类: 一些伪类只会在用户以某种方式和文档交互的时候应用。这些**用户行为伪类**，有时叫做**动态伪类**，表现得就像是一个类在用户和元素交互的时候加到了元素上一样.

    ```css
    :hover——上面提到过，只会在用户将指针挪到元素上的时候才会激活，一般就是链接元素。
    :focus——只会在用户使用键盘控制，选定元素的时候激活。
    ```

    * 伪元素: 伪元素以类似方式表现，不过表现得是像你往标记文本中加入全新的HTML元素一样，而不是向现有的元素上应用类。伪元素开头为双冒号`::`。

    ```css
    ::pseudo-element-name
    article p::first-line {
        font-size: 140%;
        font-weight: bold;
    } 
    ```

    * 伪类和伪元素组合起来

    ```css
    article p:first-child::first-line {
      font-size: 120%;
      font-weight: bold;
    }
    ```

    * 生成带有::before和::after的内容 https://cssarrowplease.com/

    ```css
    .box::before {
        content: "This should show before the other content."
    }   
    <p class="box">Content in the box in my HTML page.</p>
    ==>This should show before the other content.Content in the box in my HTML page.   
    
    ::before和::after伪元素与content属性的共同使用
    ```

    * 伪类

    | 选择器                                                       | 描述                                                         |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | [`:active`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:active) | 在用户激活（例如点击）元素的时候匹配。                       |
    | [`:any-link`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:any-link) | 匹配一个链接的`:link`和`:visited`状态。                      |
    | [`:blank`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:blank) | 匹配空输入值的[``元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input)。 |
    | [`:checked`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:checked) | 匹配处于选中状态的单选或者复选框。                           |
    | [`:current`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:current) | 匹配正在展示的元素，或者其上级元素。                         |
    | [`:default`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:default) | 匹配一组相似的元素中默认的一个或者更多的UI元素。             |
    | [`:dir`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:dir) | 基于其方向性（HTML`dir`属性或者CSS`direction`属性的值）匹配一个元素。 |
    | [`:disabled`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:disabled) | 匹配处于关闭状态的用户界面元素                               |
    | [`:empty`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:empty) | 匹配除了可能存在的空格外，没有子元素的元素。                 |
    | [`:enabled`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:enabled) | 匹配处于开启状态的用户界面元素。                             |
    | [`:first`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:first) | 匹配[分页媒体](https://developer.mozilla.org/en-US/docs/Web/CSS/Paged_Media)的第一页。 |
    | [`:first-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:first-child) | 匹配兄弟元素中的第一个元素。                                 |
    | [`:first-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:first-of-type) | 匹配兄弟元素中第一个某种类型的元素。                         |
    | [`:focus`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:focus) | 当一个元素有焦点的时候匹配。                                 |
    | [`:focus-visible`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:focus-visible) | 当元素有焦点，且焦点对用户可见的时候匹配。                   |
    | [`:focus-within`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:focus-within) | 匹配有焦点的元素，以及子代元素有焦点的元素。                 |
    | [`:future`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:future) | 匹配当前元素之后的元素。                                     |
    | [`:hover`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:hover) | 当用户悬浮到一个元素之上的时候匹配。                         |
    | [`:indeterminate`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:indeterminate) | 匹配未定态值的UI元素，通常为[复选框](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox)。 |
    | [`:in-range`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:in-range) | 用一个区间匹配元素，当值处于区间之内时匹配。                 |
    | [`:invalid`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:invalid) | 匹配诸如`<input>`的位于不可用状态的元素。                    |
    | [`:lang`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:lang) | 基于语言（HTML[lang](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/lang)属性的值）匹配元素。 |
    | [`:last-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:last-child) | 匹配兄弟元素中最末的那个元素。                               |
    | [`:last-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:last-of-type) | 匹配兄弟元素中最后一个某种类型的元素。                       |
    | [`:left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:left) | 在[分页媒体](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Pages)中，匹配左手边的页。 |
    | [`:link`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:link) | 匹配未曾访问的链接。                                         |
    | [`:local-link`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:local-link) | 匹配指向和当前文档同一网站页面的链接。                       |
    | [`:is()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:is) | 匹配传入的选择器列表中的任何选择器。                         |
    | [`:not`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:not) | 匹配作为值传入自身的选择器未匹配的物件。                     |
    | [`:nth-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-child) | 匹配一列兄弟元素中的元素——兄弟元素按照an+b形式的式子进行匹配（比如2n+1匹配元素1、3、5、7等。即所有的奇数个）。 |
    | [`:nth-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-of-type) | 匹配某种类型的一列兄弟元素（比如，`<p>`元素）——兄弟元素按照an+b形式的式子进行匹配（比如2n+1匹配元素1、3、5、7等。即所有的奇数个）。 |
    | [`:nth-last-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-last-child) | 匹配一列兄弟元素，从后往前倒数。兄弟元素按照an+b形式的式子进行匹配（比如2n+1匹配按照顺序来的最后一个元素，然后往前两个，再往前两个，诸如此类。从后往前数的所有奇数个）。 |
    | [`:nth-last-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-last-of-type) | 匹配某种类型的一列兄弟元素（比如，`<p>`元素），从后往前倒数。兄弟元素按照an+b形式的式子进行匹配（比如2n+1匹配按照顺序来的最后一个元素，然后往前两个，再往前两个，诸如此类。从后往前数的所有奇数个）。 |
    | [`:only-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:only-child) | 匹配没有兄弟元素的元素。                                     |
    | [`:only-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:only-of-type) | 匹配兄弟元素中某类型仅有的元素。                             |
    | [`:optional`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:optional) | 匹配不是必填的form元素。                                     |
    | [`:out-of-range`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:out-of-range) | 按区间匹配元素，当值不在区间内的的时候匹配。                 |
    | [`:past`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:past) | 匹配当前元素之前的元素。                                     |
    | [`:placeholder-shown`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:placeholder-shown) | 匹配显示占位文字的input元素。                                |
    | [`:playing`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:playing) | 匹配代表音频、视频或者相似的能“播放”或者“暂停”的资源的，且正在“播放”的元素。 |
    | [`:paused`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:paused) | 匹配代表音频、视频或者相似的能“播放”或者“暂停”的资源的，且正在“暂停”的元素。 |
    | [`:read-only`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:read-only) | 匹配用户不可更改的元素。                                     |
    | [`:read-write`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:read-write) | 匹配用户可更改的元素。                                       |
    | [`:required`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:required) | 匹配必填的form元素。                                         |
    | [`:right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:right) | 在[分页媒体](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Pages)中，匹配右手边的页。 |
    | [`:root`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:root) | 匹配文档的根元素。                                           |
    | [`:scope`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:scope) | 匹配任何为参考点元素的的元素。                               |
    | [`:valid`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:valid) | 匹配诸如`<input>`元素的处于可用状态的元素。                  |
    | [`:target`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:target) | 匹配当前URL目标的元素（例如如果它有一个匹配当前[URL分段](https://en.wikipedia.org/wiki/Fragment_identifier)的元素）。 |
    | [`:visited`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:visited) | 匹配已访问链接。                                             |

    * 伪元素

    | 选择器                                                       | 描述                                                 |
    | :----------------------------------------------------------- | :--------------------------------------------------- |
    | [`::after`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::after) | 匹配出现在原有元素的实际内容之后的一个可样式化元素。 |
    | [`::before`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before) | 匹配出现在原有元素的实际内容之前的一个可样式化元素。 |
    | [`::first-letter`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::first-letter) | 匹配元素的第一个字母。                               |
    | [`::first-line`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::first-line) | 匹配包含此伪元素的元素的第一行。                     |
    | [`::grammar-error`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::grammar-error) | 匹配文档中包含了浏览器标记的语法错误的那部分。       |
    | [`::selection`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::selection) | 匹配文档中被选择的那部分。                           |
    | [`::spelling-error`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::spelling-error) | 匹配文档中包含了浏览器标记的拼写错误的那部分。       |

  * **后代选择器**

    * 后代选择器——典型用单个空格（` `）字符——组合两个选择器，比如，第二个选择器匹配的元素被选择，如果他们有一个祖先（父亲，父亲的父亲，父亲的父亲的父亲，等等）元素匹配第一个选择器。选择器利用后代组合符被称作后代选择器。

    ```css
    body article p
    ```

  * **子代关系选择器**

    * 子代关系选择器是个大于号（`>`），只会在选择器选中直接子元素的时候匹配。继承关系上更远的后代则不会匹配.

    ```css
    article > p
    ```

  * **邻接兄弟**

    * 邻接兄弟选择器（`+`）用来选中恰好处于另一个在继承关系上同级的元素旁边的物件。例如，选中所有紧随`<p>`元素之后的`<img>`元素：

    ```css
    p + img
    ```

    * 常见的使用场景是， 改变紧跟着一个标题的段的某些表现方面，就像是我下面的示例那样。

  * **通用兄弟**

    * 如果你想选中一个元素的兄弟元素，即使它们不直接相邻，你还是可以使用通用兄弟关系选择器（`~`）。要选中所有的`<p>`元素后*任何地方*的`<img>`元素，我们会这样做：

    ```css
    p ~ img
    ```

  *  **使用关系选择器**

    * 你能用关系选择器，将任何在我们前面的学习过程中学到的选择器组合起来，选出你的文档中的一部分。例如如果我们想选中为`<ul>`的直接子元素的带有“a”类的列表项的话，我可以用下面的代码

    ```
    ul > li[class="a"]  {  }
    ```

    

  * **运算符**

    * 最后一组选择器可以将其他选择器组合起来，更复杂的选择元素。下面的示例用运算符（`>`）选择了`<article>`元素的初代子元素。

    ```css
    article > p { }
    ```

    | 选择器         | 示例              |
    | -------------- | ----------------- |
    | 类型选择器     | h1 { }            |
    | 通配选择器     | * { }             |
    | 类选择器       | .box { }          |
    | ID选择器       | \#unique { }      |
    | 标签属性选择器 | a[title] { }      |
    | 伪类选择器     | p:first-child { } |
    | 伪元素选择器   | p::first-line { } |
    | 后代选择器     | article p         |
    | 子代选择器     | article > p       |
    | 相邻兄弟选择器 | h1 + p            |
    | 通用兄弟选择器 | h1 ~ p            |
    
### 3.CSS盒子模型

​    

