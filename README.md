# pjax = pushState + ajax

pjax is a jQuery plugin that uses ajax and pushState to deliver a fast browsing experience with real permalinks, page titles, and a working back button.

pjax是一个jQuery插件，它通过ajax和pushState技术提供了快速的浏览体验，并且保持有效的地址、网页标题、以及浏览器的前进后退按钮。

pjax works by fetching HTML from your server via ajax and replacing the content
of a container element on your page with the loaded HTML. It then updates the
current URL in the browser using pushState. This results in faster page
navigation for two reasons:

pjax的工作是通过ajax从服务端获取HTML，在你的页面中使用加载到的HTML替换指定容器元素中的内容。然后使用pushState技术更新浏览器中的当前地址。使页面更快浏览的原因有以下两点：

* No page resources (JS, CSS) get re-executed or re-applied;
* 页面不存在资源的重复加载和应用
* If the server is configured for pjax, it can render only partial page
  contents and thus avoid the potentially costly full layout render.
  
  如果服务器端使用pjax配置，它可以只渲染局部页面内容，从而避免完整布局的渲染。

### Status of this project
### 项目的现状

jquery-pjax is **largely unmaintained** at this point. It might continue to
receive important bug fixes, but _its feature set is frozen_ and it's unlikely
that it will get new features or enhancements.

jquery-pjax的维护方向。它可能会继续重要bug的修复，但其功能是确定不变的，不会再实现新的功能和现有功能的扩展。

## Installation
## 安装

pjax depends on jQuery 1.8 or higher.
pjax 依赖于jQuery 1.8或者更高版本。

### 通过npm安装
### npm

```
$ npm install jquery-pjax
```

### standalone script
### 引入外部文件

Download and include `jquery.pjax.js` in your web page:

下载 `jquery.pjax.js` 插件并且在你的的页面中引用：

```
curl -LO https://raw.github.com/defunkt/jquery-pjax/master/jquery.pjax.js
```

## Usage
## 使用方法

### `$.fn.pjax`

The simplest and most common use of pjax looks like this:

pjax最简单常见的使用方法如下所示：

``` javascript
$(document).pjax('a', '#pjax-container')
```

This will enable pjax on all links on the page and designate the container as `#pjax-container`.

这样可以让页面上所有的a链接都实现pjax加载，并且指定 `#pajx-container`为容器。

If you are migrating an existing site, you probably don't want to enable pjax
everywhere just yet. Instead of using a global selector like `a`, try annotating
pjaxable links with `data-pjax`, then use `'a[data-pjax]'` as your selector. Or,
try this selector that matches any `<a data-pjax href=>` links inside a `<div 
data-pjax>` container:

如果你正在迁移现有的网站，你可能不希望在每个地方都使用pjax。那你可以用 `data-pjax` 标明pjax链接，然后使用 `a[data-pjax]` 作为你的选择器来代替使用 `a`的全局选择器。或者，你也可以尝试在 `<div 
data-pjax>` 容器中匹配 `<a data-pjax href=>` 链接作为这个选择器。

``` javascript
$(document).pjax('[data-pjax] a, a[data-pjax]', '#pjax-container')
```

#### Server-side configuration
#### 服务器配置

Ideally, your server should detect pjax requests by looking at the special
`X-PJAX` HTTP header, and render only the HTML meant to replace the contents of
the container element (`#pjax-container` in our example) without the rest of
the page layout. Here is an example of how this might be done in Ruby on Rails:

理论上，你的服务器应该通过查看特定的 `X-PJAX` HTTP头来检查pjax请求，并且通过加载到的HTML替换容器元素（在我们的例子中是 `#pajx-container`）的内容来渲染，没有其余的页面布局。下面的例子是在Ruby on Rails中的做法：

``` ruby
def index
  if request.headers['X-PJAX']
    render :layout => false
  end
end
```

If you'd like a more automatic solution than pjax for Rails check out [Turbolinks][].

如果你想要了解比上述方法更自动化的方案，请查看[Turbolinks][]。

[Check if there is a pjax plugin][plugins] for your favorite server framework.

[看看是不是在这些pjax插件][plugins]中能找到你喜欢的服务端框架匹配的。

Also check out [RailsCasts #294: Playing with PJAX][railscasts].

也可以看看[RailsCasts #294: Playing with PJAX][railscasts].

#### Arguments
#### 参数

The synopsis for the `$.fn.pjax` function is:
`$.fn.pjax` 方法的概述：

``` javascript
$(document).pjax(selector, [container], options)
```

1. `selector` is a string to be used for click [event delegation][$.fn.on].
1. `selector` 是用于click事件的选择器。
2. `container` is a string selector that uniquely identifies the pjax container.
2. `container` 是具有唯一标识的pjax容器选择器。
3. `options` is an object with keys described below.
3. `options` 是有下列选项的对象。

##### pjax options
##### pjax配置选项

选项 | 默认 | 描述
----|---------|------------
`timeout` | 650 | ajax超时之后强制刷新整个页面
`push` | true | 使用 [pushState][] 在浏览器中添加历史记录
`replace` | false | 替换URL地址但不添加浏览器历史记录
`maxCacheLength` | 20 | 以前容器内容的最大缓存值
`version` | | 通过字符串或者方法返回当前pjax版本
`scrollTo` | 0 | 浏览器的纵向滚动位置。通过`false`避免改变滚动位置
`type` | `"GET"` | 看[$.ajax][]
`dataType` | `"html"` | 看 [$.ajax][]
`container` | | 要被替换内容元素的CSS选择器
`url` | link.href | 通过一个字符串或者方法返回ajax请求响应的URL
`target` | link | [pjax events](#events)最终的值`relatedTarget`
`fragment` | | 提取ajax响应内容碎片的CSS选择器

You can change the defaults globally by writing to the `$.pjax.defaults` object:

你可以使用 `$.pjax.defaults` 对象在全局改变默认配置：

``` javascript
$.pjax.defaults.timeout = 1200
```

### `$.pjax.click`

This is a lower level function used by `$.fn.pjax` itself. It allows you to get a little more control over the pjax event handling.

将`$.fn.pjax` 作为一个基础的方法使用。它可以让你操作更多的事件行为。

This example uses the current click context to set an ancestor element as the container:

这个列子使用当前的click上下文来设置一个祖先元素作为容器：

``` javascript
if ($.support.pjax) {
  $(document).on('click', 'a[data-pjax]', function(event) {
    var container = $(this).closest('[data-pjax-container]')
    var containerSelector = '#' + container.id
    $.pjax.click(event, {container: containerSelector})
  })
}
```

**NOTE** Use the explicit `$.support.pjax` guard. We aren't using `$.fn.pjax` so we should avoid binding this event handler unless the browser is actually going to use pjax.

**注意** 明确使用 `$.support.pjax` 控制。我们不使用`$.fn.pjax`，我们应该避免绑定这个事件动作，除非浏览器正在使用pjax。

### `$.pjax.submit`

Submits a form via pjax.

通过pjax提交表单。

``` javascript
$(document).on('submit', 'form[data-pjax]', function(event) {
  $.pjax.submit(event, '#pjax-container')
})
```

### `$.pjax.reload`

Initiates a request for the current URL to the server using pjax mechanism and replaces the container with the response. Does not add a browser history entry.

使用pjax机制发起一个当前URL的请求到服务器，并且通过响应内容替换容器元素。不添加浏览器历史记录。

``` javascript
$.pjax.reload('#pjax-container', options)
```

### `$.pjax`

Manual pjax invocation. Used mainly when you want to start a pjax request in a handler that didn't originate from a click. If you can get access to a click `event`, consider `$.pjax.click(event)` instead.

pjax手动调用。主要用于当不是来自于click事件时，你想发起一个pjax请求处理的情况。如果你能获得一个click `event`。可以考虑通过`$.pjax.click(event)`代替。

``` javascript
function applyFilters() {
  var url = urlForFilters()
  $.pjax({url: url, container: '#pjax-container'})
}
```

## Events
## 事件

All pjax events except `pjax:click` & `pjax:clicked` are fired from the pjax
container element.

除了 `pjax:click` 和 `pjax:clicked`，其他所有pjax事件都是在pjax容器元素上触发的。

<table>
<tr>
  <th>事件</th>
  <th>关闭</th>
  <th>参数</th>
  <th>说明</th>
</tr>
<tr>
  <th colspan=4>pjax链接的事件生命周期如下</th>
</tr>
<tr>
  <td><code>pjax:click</code></td>
  <td>✔︎</td>
  <td><code>options</code></td>
  <td>从一个链接触发激活到关闭阻止</td>
</tr>
<tr>
  <td><code>pjax:beforeSend</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>设置XHR 头</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:send</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:clicked</code></td>
  <td></td>
  <td><code>options</code></td>
  <td>一个pjax链接一个点击之后触发</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>从服务器加载的内容替换HTML片段之前</td>
</tr>
<tr>
  <td><code>pjax:success</code></td>
  <td></td>
  <td><code>data, status, xhr, options</code></td>
  <td>从服务器加载的内容替换HTML片段之后</td>
</tr>
<tr>
  <td><code>pjax:timeout</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td> <code>options.timeout</code>触发之后; 强制刷新，除非被关闭</td>
</tr>
<tr>
  <td><code>pjax:error</code></td>
  <td>✔︎</td>
  <td><code>xhr, textStatus, error, options</code></td>
  <td>ajax出错；强制刷新，除非被关闭</td>
</tr>
<tr>
  <td><code>pjax:complete</code></td>
  <td></td>
  <td><code>xhr, textStatus, options</code></td>
  <td>无论结果如果，总在ajax响应后触发</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <th colspan=4>浏览器前进后退按钮相关事件</th>
</tr>
<tr>
  <td><code>pjax:popstate</code></td>
  <td></td>
  <td></td>
  <td>事件 <code>重定向</code> 特性: &quot;前进&quot;/&quot;后退&quot;</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>内容替换之前</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>用缓存内容正确替换HTML片段时</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>替换内容之后</td>
</tr>
<tr>
  <td><code>pjax:callback</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>页面脚本加载完成后（admui项目）</td>
</tr>
</table>

**NOTE**
使用Admui项目的用户，不要替换其他地方获取的pjax插件。对pjax插件Admui做了一些自己的处理。

`pjax:send` & `pjax:complete` are a good pair of events to use if you are implementing a
loading indicator. They'll only be triggered if an actual XHR request is made,
not if the content is loaded from cache:

如果你正在执行加载操作，`pjax:send`和`pjax:complete`对你来说是很有用的一对事件。它们只有在是实际XHR请求时才会被触发，而不是从缓存中加载内容时：

``` javascript
$(document).on('pjax:send', function() {
  $('#loading').show()
})
$(document).on('pjax:complete', function() {
  $('#loading').hide()
})
```

An example of canceling a `pjax:timeout` event would be to disable the fallback
timeout behavior if a spinner is being shown:
下面是关闭 `pjax:timeout` 事件的例子，将禁用超时后的默认行为：

``` javascript
$(document).on('pjax:timeout', function(event) {
  // Prevent default timeout redirection behavior
  // 阻止默认的超时重定向行为
  event.preventDefault()
})
```

## Advanced configuration
## 高级配置

### Reinitializing plugins/widget on new page content
### 在一个新页面中重新预置插件或工具

The whole point of pjax is that it fetches and inserts new content _without_
refreshing the page. However, other jQuery plugins or libraries that are set to
react on page loaded event (such as `DOMContentLoaded`) will not pick up on
these changes. Therefore, it's usually a good idea to configure these plugins to
reinitialize in the scope of the updated page content. This can be done like so:

pjax的特点是它不会刷新页面即可获取并插入新内容。但是，其他jQuery插件或库为页面内容绑定了加载事件（如DOMContentLoaded）的，此时不会在响应。 因此，在更新的页面内容的范围内重新初始化插件是一个通用的方法。 我们可以这样做：

``` js
$(document).on('ready pjax:end', function(event) {
  $(event.target).initializeMyPlugin()
})
```

This will make `$.fn.initializeMyPlugin()` be called at the document level on
normal page load, and on the container level after any pjax navigation (either
after clicking on a link or going Back in the browser).


这就可以让`$ .fn.initializeMyPlugin()`在正常页面加载和pjax加载时（点击链接或浏览器前进后退之后）都能被调用。

### Response types that force a reload

### 强制重载的响应类型

By default, pjax will force a full reload of the page if it receives one of the
following responses from the server:

* Page content that includes `<html>` when `fragment` selector wasn't explicitly
  configured. Pjax presumes that the server's response hasn't been properly
  configured for pjax. If `fragment` pjax option is given, pjax will extract the
  content based on that selector.

* Page content that is blank. Pjax assumes that the server is unable to deliver
  proper pjax contents.

* HTTP response code that is 4xx or 5xx, indicating some server error.

默认情况下，如果pjax从服务器收到以下响应之一，则强制重新加载页面：

* 页面包含`<html>`标签，没有明确指定`fragment`选择器时。 Pjax就会推测服务器的响应没有被正确配置为pjax。 如果配置了 `fragment` pjax选项，pjax将根据该选择器提取内容。

* 页面内容是空白的。 Pjax就会推测服务器无法提供正确的pjax内容。

* HTTP响应代码为4xx或5xx，表示某些服务器错误。

### Affecting the browser URL

### 浏览器URL变动

If the server needs to affect the URL which will appear in the browser URL after
pjax navigation (like HTTP redirects work for normal requests), it can set the
`X-PJAX-URL` header:

如果服务器需要影响pjax加载后的浏览器地址栏中显示的URL（例如HTTP重定向适用于正常请求），则可以设置X-PJAX-URL头：

``` ruby
def index
  request.headers['X-PJAX-URL'] = "http://example.com/hello"
end
```

### Layout Reloading
### 重载布局

Layouts can be forced to do a hard reload when assets or html changes.

静态资源或页面改变时，布局将被强制进行重载

First set the initial layout version in your header with a custom meta tag.

首先，用一个自定义的meta标签在你的头部初始化layout版本。

``` html
<meta http-equiv="x-pjax-version" content="v123">
```

Then from the server side, set the `X-PJAX-Version` header to the same.

然后在服务端设置相同的`X-PJAX-Version`头。

``` ruby
if request.headers['X-PJAX']
  response.headers['X-PJAX-Version'] = "v123"
end
```

Deploying a deploy, bumping the version constant to force clients to do a full reload the next request getting the new layout and assets.

部署实现，版本不同之后会强制整个页面重载，新的请求会获取新的布局和相关资源。


[$.fn.on]: http://api.jquery.com/on/
[$.ajax]: http://api.jquery.com/jQuery.ajax/
[pushState]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history#Adding_and_modifying_history_entries
[plugins]: https://gist.github.com/4283721
[turbolinks]: https://github.com/rails/turbolinks
[railscasts]: http://railscasts.com/episodes/294-playing-with-pjax
