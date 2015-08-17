# 文档(Docs)

我们为`Aurelia`框架准备了一个非常丰富的计划文档.不幸的是,我们还没有完成。然而,对于早期预览期间,我们把文档组织在一块,包含了你或许想要执行的最常见的例子。如果你有问题,我们希望你能加入我们(gitter频道)(https://gitter.im/aurelia/discuss)。

> **注意:** 寻找当前文档的其他语言版本? 看看我们的[文档仓库](https://github.com/aurelia/documentation).

<h2 id="browser-support"><a href="#browser-support">浏览器支持</a></h2>  

Aurelia起初是为常见的浏览器设计的. 包括 `Chrome`,`Firefox`,`IE11`和`Safari 8`.然而,我们指明了怎么去支持`IE9`和以上版本. 为了让它起作用, 你需要为`mutationobservers`添加另外的`polyfill`支持. 这个可以通过`jspm`安装`github:polymer/mutationobservers`. 接着像下面为`aurelia-bootstrapper`包装调用方式.

```markup
<script src="jspm_packages/system.js"></script>
<script src="config.js"></script>
<script>
  // Loads WeakMap polyfill needed by MutationObservers
  System.import('core-js').then( function() {
    // Imports MutationObserver polyfill
    System.import('polymer/mutationobservers').then( function() {
      // Ensures start of Aurelia when all required IE9 dependencies are loaded
      System.import('aurelia-bootstrapper');
    })
  });
</script>
```


> **注意:** `WeakMap`不是Aurelia本身需要,而是`MutationObserver polyfill`要使用.


<h2 id="startup-and-configuration"><a href="#startup-and-configuration">开始 & 配置(Startup & Configuration)</a></h2>

很多代码执行平台有一个`main`或者一个入口. Aurelia也是这样. 如果你已经阅读过[Get Started](/get-started.html) 页面, 离就会发现`aurelia-app`属性. 就这样简单的放置到HTML元素上后Aurelia引导程序就会加载 _app.js_ 和 _app.html_, 将他们绑定到一起,并且将他们注入到你放置那个属性的DOM元素上.

时常你想要配置框架,或者执行一些置前的代码去展示任何用户想要的.所以,变化是,随着你项目的进展,你将朝着需要一些启动配置迁移. 为了这样做, 你可以为`aurelia-app`属性提供一个值以便指向配置模块. 这个模块应当导出一个单一的命名为`configure`的`function`, 传递它一个Aurelia对象参数,以便于你可以通过它配置框架决定怎样,何时,何地展示你的UI. 下面是一个配置例子:

```javascript
import {LogManager} from 'aurelia-framework';
import {ConsoleAppender} from 'aurelia-logging-console';

LogManager.addAppender(new ConsoleAppender());
LogManager.setLevel(LogManager.logLevel.debug);

export function configure(aurelia) {
  aurelia.use
    .defaultBindingLanguage()
    .defaultResources()
    .history()
    .router()
    .eventAggregator()
    .plugin('custom-plugin');

  aurelia.start().then(a => a.setRoot('app', document.body));
}
```

通过特别的自定义插件, 这段代码是`aurelia-app`正常为你做的本质. 当你改变配置文件的方式, 你需要自己配置这些, 然而你也可以安装自定义插件, 设置附带一些服务(`services`)的依赖注入容器, 并且安装在视图模板(`view templates`)中使用的全局资源.

如果你想要改变启动配置文件的方式, 你实际上可以写一个总结全部上面提到的标准配置项的简单文件, 这儿是一个要看到的例子: 

```javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  aurelia.start().then(a => a.setRoot());
}
```

<h3 id="logging"><a href="#logging">日志(Logging)</a></h3>

Aurelia有一个简单的框架自己使用的日志抽象. 默认它是没有操作的. 上面的配置展示了怎样安装一个获取日志数据并且输出它到控制台的输出源(`appender`). 你也可以设置日志的输出级别. `logLevel`可能的枚举值包括: `none`, `error`, `warn`, `info` 和 `debug`.

你可以方便的创建自己的日志输出源, 简单的实现一个匹配日志输出源接口的类. 最好的方式是去看看我们的实现 [console log appender's source](https://github.com/aurelia/logging-console/blob/master/src/index.js).

<h3 id="plugins"><a href="#plugins">插件(Plugins)</a></h3>

一个 _插件_ 仅仅就是一个导出`configure`函数的模块. 在启动期间Aurelia会加载所有的插件模块然后调用它们的`configure`函数, 传递给它们`FrameworkConfiguration`实例以便它们可以适当的配置框架, 插件可以可选的从它们的`configure`函数返回一个`Promise`对象以便执行异步配置任务. 当写一个插件, 要确保明确的提供全部的元数据(`metadata`), 包括一个为了自定义元素的视图策略(`View Strategy`).

为了在你的应用中的插件中配置你可以指定一个函数或者对象作为`configure`函数的第二个参数. 在你的安装工作结束你插件的配置函数可以使用. 你插件的配置可以像这样写:

```javascript
aurelia.use.plugin('plugin-name', config => { /* configuration work */ });
```

> **注意:** 不要在插件内依赖命名约定. 因为你不清楚插件的使用者会怎样改变Aurelia的约定. 第三方的插件应该明确为了它们的功能在不同的上下文中正确.

<h4 id="features"><a href="#features">特性插件(Feature Plugins)</a></h4>

上面的插件接口(`API`)是为安装外部的三方的插件设计的. 然而, 我或许想组织你自己项目里的一系列插件, 为了这样做, 为你自己的内部`功能`(`feature`)插件简单的创建一个文件夹, 在文件夹中创建一个`index.js`文件导出一个单一的`configure`函数, 这个函数像三方插件一样工作, 在启动的时候注册你的功能, 你将使用下面的代码:

```javascript
aurelia.use.feature('feature-folder-name-here');
```

<h4 id="promises"><a href="#promises">许诺(Promises)</a></h4>

默认的, Aurelia使用ES6原生的许诺(`Promises`)或者一个垫片(`polyfill`). 然而,你可以用优秀的 [Bluebird](https://github.com/petkaantonov/bluebird) 许诺类库来替代. 简单的在你的页面中包含它在引用其它类库之前, 这将提供自己的符合标准的许诺实现,它比原生的更快并且有更好的调试(`debugging`)支持. 另外, 当结合`Babel`转码器使用时, 为了改进异步编码你可以使用[coroutines](http://babeljs.io/docs/usage/transformers/other/bluebird-coroutines/) .

<h3 id="framework-configuration"><a href="#framework-configuration">框架配置(Framework Configuration)</a></h3>

一个`Aurelia`实例对象被传递给你主要的`configure`函数像上面的例子展示. 这个对象的`use`属性是一个可以用作准备框架的插件(`plugins`),功能(`features`),全局资源(`global resources`),服务(`services`)和其他更多的`FrameworkConfiguration`实例. 并且, 插件和功能`configure`函数接受一个`FrameworkConfiguration`实例参数以便它们可以配置自身. 因为这个配置API很重要, 我们在下面的代码中提供一个简要的解释.

```javascript
export class FrameworkConfiguration {
  /**
   * Adds an existing object to the framework's dependency injection container.
   */
  instance(type: any, instance: any): FrameworkConfiguration;

  /**
   * Adds a singleton to the framework's dependency injection container.
   */
  singleton(type: any, implementation?: Function): FrameworkConfiguration;

  /**
   * Adds a transient to the framework's dependency injection container.
   */
  transient(type: any, implementation?: Function): FrameworkConfiguration;

  /**
   * Adds an async function that runs before the plugins are run.
   */
  preTask(task: Function): FrameworkConfiguration;

  /**
   * Adds an async function that runs after the plugins are run.
   */
  postTask(task: Function): FrameworkConfiguration;

  /**
   * Configures an internal feature plugin before Aurelia starts.
  */
  feature(plugin: string, config: any): FrameworkConfiguration;

  /**
   * Adds globally available view resources to be imported into the Aurelia framework.
   */
  globalResources(resources: string | string[]): FrameworkConfiguration;

  /**
   * Renames a global resource that was imported.
   */
  globalName(resourcePath: string, newName: string): FrameworkConfiguration;

  /**
   * Configures an external, 3rd party plugin before Aurelia starts.
   */
  plugin(plugin: string, config: any): FrameworkConfiguration;

  /**
   * Plugs in the default binding language from aurelia-templating-binding.
   */
  defaultBindingLanguage(): FrameworkConfiguration;

  /**
   * Plugs in the router from aurelia-templating-router.
   */
  router(): FrameworkConfiguration;

  /**
   * Plugs in the default history implementation from aurelia-history-browser.
   */
  history(): FrameworkConfiguration;

  /**
   * Plugs in the default templating resources (if, repeat, show, compose, etc.) from aurelia-templating-resources.
   */
  defaultResources(): FrameworkConfiguration;

  /**
   * Plugs in the event aggregator from aurelia-event-aggregator.
   */
  eventAggregator(): FrameworkConfiguration;

  /**
   * Sets up the Aurelia configuration. This is equivalent to calling `.defaultBindingLanguage().defaultResources().history().router().eventAggregator();`
   */
  standardConfiguration(): FrameworkConfiguration;

  /**
   * Plugs in the ConsoleAppender and sets the log level to debug.
   */
  developmentLogging(): FrameworkConfiguration;
}
```

<h2 id="views-and-view-models"><a href="#views-and-view-models">视图和视图模型(Views and View Models)</a></h2>

在Aurelia中, 用户接口元素是 _view_ 和 _view-model_ 成对结合的.  _view_ 是HTML编写, 会被渲染到DOM.  _view-model_ 是JavaScript编写, 提供到 _view_ 的数据和行为. 模板引擎和依赖注入(`DI`)负责创建这些视图和视图模型对, 增强处理过程中的可预见的生命周期. 一旦实例化, Aurelia强大的 _databinding_ 连接这两个部件在一起, 允许在你数据的变化反映到 _view_ ,反之亦然. 这种关注点的分离对开发者和设计者的合作是很赞的. 可维护性, 结构灵活, 甚至源码控制.

<h3 id="dependency-injection"><a href="#dependency-injection">依赖注入(Dependency Injection (DI))</a></h3>

视图模型和其他的接口元素, 例如自定义元素和自定义属性, 以类`classes`的形式创建, 被框架使用依赖注入容器实例化. 用这种编码书写很容易模块化和测试. 相比较创建庞大的类(`classes`), 你可以分割成小的对象合作去达到一个目标. 依赖注入可以在运行时(`runtime`)把这些部件组织在一起. 

为了使用依赖注入, 你简单的装饰你的类,去告知框架什么应该传递到它的构造函数`constructor`. 这里是一个视图模型的简单例子, 它依赖Aurelia的`fetch client`.

```javascript
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(HttpClient)
export class CustomerDetail{
    constructor(http){
        this.http = http;
    }
}
```

如果`ES2016`和`TypeScript`装饰器开启的话, 你仅仅添加`inject`装饰器, 每个注入类型传递一个参数. 如果没有使用支持装饰(`Decorators`)的开发语言, 或者就不想去使用, 你也可以添加一个静态(`static`)的属性或者方法到你的命名为`inject`的类. 它必须返回一个可注入类型的数组. 这里是一个简单的`CoffeeScript`写的`CommonJS modules`的例子.

```coffeescript
HttpClient = require('aurelia-fetch-client').HttpClient;

class Flickr
  constructor: (@http) ->
  @inject:[HttpClient]
```

如果你正在使用`TypeScript`, 你可以使用 `--emitDecoratorMetadata` 编译标志位伴随着Aurelia的`@autoinject`装饰器,允许框架读取标准的`TS`类型信息. 这样的话, 就没有必要去重复类型. 这里是看起来的样子:

```javascript
import {autoinject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@autoinject
export class CustomerDetail {
    constructor(public http:HttpClient) {
        this.http = http;
    }
}
```

> **注意:** 这里`TypeScript`实现编译选项的方式有一个有意思的细节. 它实际上可以和其他装饰器一起工作. 所以, 如果你已经在你的`TS`类使用其他装饰器, 就没有必要再包含 `autoinject` 装饰器. 类型信息仍然会被Aurelia的依赖注入矿建自动发现.

但明确声明依赖项, 重要的是要知道它们不需要必须是构造类型, 它们也可以是`resolvers`实例. 例如, 看看这里: 

```javascript
import {Lazy, inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(Lazy.of(HttpClient))
export class CustomerDetail{
    constructor(getHTTP){
        this.getHTTP = getHTTP;
    }
}
```

`Lazy`解析器实际上不提供`HttpClient`的实例, 作为替代, 它提供一个函数, 当被调用, 它会返回一个`HttpClient`的实例. 有几个不同的解析器可以拆箱使用, 并且你可以创建自己的解析器,通过授权一个类继承`Resolver`. 这里是我们为你提供的一个列表:

* `Lazy` - 注入一个函数,会延迟计算评估依赖.
    * ex. `Lazy.of(HttpClient)`
* `All` - 注入一个提供键值注册的全部服务的数组.
    * ex. `All.of(Plugin)`
* `Optional` - 注入一个类的实例当它已经存在于容器中, 反之注入null.
    * ex. `Optional.of(LoggedInUser)`

除了这些解析器外, 你也可以使用 `Registration` 装饰器去特别指定默认的登记, 或者一个实例的生命周期. 默认的, 依赖注入容器假定一切都是单例模式; 整个app就一个实例. 然而, 你可以使用注册装饰器去改变默认行为, 这里是一个例子:

```javascript
import {transient, inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@transient()
@inject(HttpClient)
export class CustomerDetail{
    constructor(http){
        this.http = http;
    }
}
```

现在, 每次依赖注入容器被请求一个`CustomerDetail`实例, 容器将会返回一个新的实例, 而不是一个单例. `singleton` 和 `transient` 注册方式都系统提供. 但是你可以创建你自己的通过写一个类实现`register(container, key, fn)`方法. 接着, 简单的添加它的实例到这个类通过附带`registration`装饰器.

如果你不能或者不想使用装饰器, 别担心, 我们提供一个备案机制. 简单的提供一个静态的 `decorators` 属性或者方法, 然后使用链式的`Decorators`助手. 这个助手拥有我们全部的装饰器的方法. 所以,对于我们在任何开发语言中都可以很简单的使用. 这里展示的是如何用`CoffeeScript`语言编写上面的例子:

```coffeescript
HttpClient = require('aurelia-http-client').HttpClient;
Decorators = require('aurelia-framework').Decorators;

class CustomerDetail
  constructor: (@http) ->
  @decorators:Decorators.transient().inject(HttpClient);
```

<h3 id="parent-vm-reference"><a href="#parent-vm-reference">父级视图模型(Parent View Models)</a></h3>

默认的一个视图模型(`View-model`)的类是限制到被注入的对象中和它的子类一样.有时, 引用一个父视图模型中的对象和方法是值得的, 这样能够达到通过储存父模型在视图生命周期的 _bind_ 方法中.

```javascript
class ChildViewModel {
  bind(bindingContext) {
    this.$parent = bindingContext;
  }
}
```

<h2 id="templating"><a href="#templating">模板(Templating)</a></h2>

Aurelia的模板引擎是负责加载力的视图和他们所需的资源, 编译你的HTML为了理想的性能和渲染你的UI到屏幕. 创建一个视图, 全部你所要做的就是创建一个HTML文件,里面要包含一个`HTMLTemplate`, 这里是一个简单的例子:

```markup
<template>
    <div>Hello World!</div>
</template>
```

所有在`template`标签你的内容都将被Aurelia管理. 然后因为Aurelia使用`HTMLImport`技术加载视图, 你当然也可以包含`links`, 这样它们会被适当的加载, 包含语义上的相对的资源. 换句话说, 你可以这样做: 

```markup
<link rel="stylesheet" href="hello.css">

<template>
    <div class="hello">Hello World!</div>
</template>
```

这样允许你动态的加载每个视图的样式表文件, 甚至于运行时的web组件.

任何时候当你需要一个Aurelia特别指定的资源, 例如一个Aurelia的 _Custom Element_, _Custom Attribute_ 或者 _Value Converter_,你应该替代的使用一个`require`元素在你的视图中. 这里有一个例子:

```markup
<template>
  <require from='nav-bar'></require>

  <nav-bar router.bind="router"></nav-bar>

  <div class="page-host">
    <router-view></router-view>
  </div>
</template>
```

这个 `nav-bar` 样例是一个我们需要使用的Aurelia的 _Custom Element_ . 通过使用Aurelia的`require`元素引起框架资源管道式的处理被导入的项目, 这样会有以下好处:

* 防重(Deduping) - 在整个app资源只下载一次. 甚至于其它视图也需要同样的元素, 它将不会再次下载.
* 一次性编译(One-time Compilation) - 自定义元素的模板需要这种方式被一次编译对于整个应用.
* 本地作用域(Local Scope) - 需要的资源仅仅在需要他的视图中可见, 减少名称冲突的可能性,并且提升了可维护性和易懂的特性通过消除全局性.
* 重命名(Renaming) - 资源能够被重命名当在被两方三方资源通过相同的或者相似的名称在同样的视图中被使用时被应用.
    - ex. `<require from="nav-bar" as="foo-bar"></require>` - Now instead of using a `nav-bar` element you can use a `foo-bar` element. (This is based on ES6 import syntax where renaming is considered a replacement for using an Alias because it strictly renames the type locally.)

* 封装(Packages) - 需要的资源可以指向一个拥有多个全部将被导入到相同视图的资源模块.
* 可扩展性(Extensibility) - 你可以顶一个新的资源类型, 当像这种方式被需要, 能够执行自定义加载(async one-time)并且注册(once per-view). 这是声明式,可扩展式的资源加载管道.
* ES6 - 代码通过ES6被加载, 而不是`HTMLImport`加载机制, 允许全部的特性和你加载方式的扩张性. 这种设计选择完全的统一了所有app资源的加载, 不管是JavaScript或者HTML.

在你的视图中和数据绑定一样你会经常使用不同类型的像上面提到的资源.

> ** 注意:** 你或许觉得不得不导入资源到每个视图会很乏味, 在启动引导的阶段里可以配置Aurelia的全局资源对每个视图可见. 仅仅使用 `aurelia.globalizeResources(...resourcePaths)`.

Aurelia垫片浏览器不支持模板. 然而, 一些模板的特性不能被填充垫片, 需要特性的环境. 特别要提的是这发生在添加`<template>`元素到 `<select>` 和 `<table>` 元素中. 接下来的例子不能在不能原生支持模板的浏览器中使用:

```markup
  <table>
    <template repeat.for="customer of customers">
      <tr>
        <td>${customer.fullName}</td>
      </tr>
    </template>
  </table>
```

为了循环重复`<tr>`元素, 可以简单的添加`repeat`到`<tr>`自己的身上.

```markup
  <table>
    <tr repeat.for="customer of customers">
      <td>${customer.fullName}</td>
    </tr>
  </table>
```

<h3 id="databinding"><a href="#databinding">数据绑定(Databinding)</a></h3>

数据绑定(`Databinding`)允许你链接状态和行为在一个JavaScript对象和一个HTML视图中. 当这种链接被建立, 任何链接属性的改变会被单向或者双向的同步. 在JavaScript对象中的变化能够被反映到视图, 并且在视图中的变化会被反映到JavaScript对象中. 建立这种链接, 你需要在HTML中使用"binding commands", 绑定命令通过使用中绑定操作符`"."`清楚的的被识别. 任何时候HTML属性包含一个`"."`, 编译器会传递属性名和值给绑定开发语言去解析. 结果就是一个或者更多绑定表达式有能力的建立这种链接关系在视图创建后.

你可以用自己的绑定命令扩展系统指令, 然而,Aurelia已经提供了一系列的指令覆盖了大多常用的使用情形.

<h4 id="binding-modes"><a href="#binding-modes">绑定, 单向, 双向, 一次性(bind, one-way, two-way & one-time)</a></h4>

最常见的绑定命令是`.bind`. 这将导致的结果是,除了表单(`form`)元素会使用双向(`"two-way"`)绑定外, 其他全部的属性绑定将会使用单向(`"one-way"`)绑定.

_What does this mean though?_

One-way binding means that changes flow from your JavaScript view-models into the view, not from the view into the view-model. Two-way binding means that changes flow in both directions. `.bind` attempts to use a sensible default by assuming that if you are binding to a form element's value property then you probably wish the changes made in the form to flow into your view-model. For everything else it uses one-way binding, especially since, in many cases, two-way binding to non-form elements would be nonsensical. Here's a small binding example using `.bind`:

```markup
<input type="text" value.bind="firstName">
<a href.bind="url">Aurelia</a>
```

In the above example, the `input` will have its `value` bound to the `firstName` property on the view-model. Changes in the `firstName` property will update the `input.value` and changes in the `input.value` will update the `firstName` property. On the other hand, the `a` tag will have its `href` bound to the `url` property on the view-model. Only changes in the `url` property will flow into the `href` of the `a` tag, not the other way.

You can always be explicit and use `.one-way` or `.two-way` in place of `.bind` though. A common case where this is required is with Web Components that function as input-type controls. So, you can imagine doing something like this:

```markup
<markdown-editor value.two-way="markdown"></markdown-editor>
```

In order to optimize performance and minimize CPU and memory usage, you can alternatively leverage the `.one-time` binding command to flow data from the view-model into the view "one time". This will happen during the initial binding phase, after which, no synchronization will occur.

<h4 id="event-modes"><a href="#event-modes">delegate, trigger & call</a></h4>

Binding commands don't only connect properties and attributes, but can be used to trigger behavior. For example, if you want to invoke a method on the view-model when a button is clicked, you would use the `trigger` command like this:

```markup
<button click.trigger="sayHello()">Say Hello</button>
```

When the button is clicked, the `sayHello` method on the view-model will be invoked. That said, adding event handlers to every single element like this isn't very efficient, so often times you will want to use event delegation. To do that, use the `.delegate` command. Here's the same example but with event delegation instead:

```markup
<button click.delegate="sayHello()">Say Hello</button>
```

The `$event` property can be passed as an argument to a delegate/trigger function call if you need to access the event object.

```markup
<button click.delegate="sayHello($event)">Say Hello</button>
```

> **Note:** If you aren't familiar with event delegation, it's a technique that uses the bubbling nature of DOM events. When using `.delegate`, a single event handler is attached to the document, rather than on each element. When the element's event is fired, it bubbles up the DOM until it reaches the document, where it is handled. This is a more memory efficient way of handling events and it's recommended to use this as your default mechanism.

> **Note:** Event delegation does not work from inside a *closed* ShadowDOM. It will work from within an open ShadowDOM with no trouble though.

All of this works against DOM events in some way or another. Occasionally you may have an Aurelia Custom Attribute or Element that wants a reference to your function directly so that it can invoke it manually at a later time. To pass a function reference, use the `.call` binding (since the attribute will _call_ it later):

```markup
<button touch.call="sayHello()">Say Hello</button>
```

Now the Custom Attribute `touch` will get a function that it can call to invoke your `sayHello()` code. Depending on the nature of the implementor, you may be able to receive data from the caller. This works the same as with trigger/delegate by providing an `$event` object.

<h4 id="string-interpolation"><a href="#string-interpolation">string interpolation</a></h4>

Sometimes you need to bind properties directly into the content of the document or interleave them within an attribute value. For this, you can use the string interpolation syntax `${expression}`. String interpolation is a one-way binding, the output of which is converted to a string. Here's an example:

```markup
<span>${fullName}</span>
```

The `fullName` property will be interpolated directly into the span's content. You can also use this to handle css class bindings like so:

```markup
<div class="dot ${color} ${isHappy ? 'green' : 'red'}"></div>
```

In this snippet "dot" is a statically present class and "green" is present only if `isHappy` is true, otherwise the "red" class is present. Additionally, whatever the value of `color` is...that is added as a class.

> **Note:** You can use simple expressions inside your bindings. Don't try to do anything too fancy. You don't want code in your view. You only want to establish the linkage between the view and its view-model.

<h4 id="ref"><a href="#ref">ref</a></h4>

In addition to commands and interpolation, the binding language recognizes the use of a special attribute: `ref`. By using `ref` you can create a local name for an element which can then be referenced in another binding expression. It will also be set as a property on the view-model, so you can access it through code. Here's a neat example of using `ref`:

```markup
<input type="text" ref="name"> ${name.value}
```

You can also use `ref` as a binding command to get the view-model instance that backs an Aurelia Custom Element or Custom Attribute. By using this technique, you can connect different components to each other:

```markup
<producer producer.ref="producerVM"></producer>
<consumer input.bind="producerVM.output"></consumer>
```

`producer.ref="producerVM"` creates an alias to the view-model for the `producer` custom element which you can then use elsewhere to pass to another custom element or to use properties of the VM. Thus in the second line of the code above, `consumer` has a property called `input` that has been bound to the `output` property of the VM of the `producer`. There are a few ways to use `ref` to reference elements and view-models:

* `attribute-name.ref="someIdentifier"`- create a reference to a custom attribute's class instance
* `element-name.ref="someIdentifier"`- create a reference to a custom element's class instance
* `ref="someIdentifier"` - create a reference to the HTMLElement in the DOM

<h4 id="select-elements"><a href="#select-elements">select elements</a></h4>

`value.bind` on an HTMLSelectElement has special behavior to support the element's single and multi-select modes as well as binding to objects.

A typical select element is rendered using a combination of `value.bind` and `repeat`, like this:

```markup
<select value.bind="favoriteColor">
    <option>Select A Color</option>
    <option repeat.for="color of colors" value.bind="color">${color}</option>
</select>
```

Sometimes you want to work with object instances rather than strings.  Here's the markup for building a select element from a theoretical array of employee objects:

```markup
<select value.bind="employeeOfTheMonth">
  <option>Select An Employee</option>
  <option repeat.for="employee of employees" model.bind="employee">${employee.fullName}</option>
</select>
```

The primary difference between this example and the previous example is we're storing the option values in a special property, `model`, instead of the option element's `value` property which only accepts strings.

<h4 id="multi-select-elements"><a href="#multi-select-elements">multi select elements</a></h4>

You can bind the select element's value to an array property in multi-select scenarios.  Here's how you'd bind an array of strings, `favoriteColors`:

```markup
<select value.bind="favoriteColors" multiple>
    <option repeat.for="color of colors" value.bind="color">${color}</option>
</select>
```

This works with arrays of objects as well:

```markup
<select value.bind="favoriteEmployees" multiple>
  <option repeat.for="employee of employees" model.bind="employee">${employee.fullName}</option>
</select>
```

<h4 id="radios"><a href="#radios">radios</a></h4>

`checked.bind` on an HTMLInputElement has special behavior to support binding non-boolean values such as strings and objects.

A typical radio button group is rendered using a combination of `value.bind` and `repeat`, like this:

```markup
<label repeat.for="color of colors">
  <input type="radio" name="clrs" value.bind="color" checked.bind="$parent.favoriteColor" />
  ${color}
</label>
```

Sometimes you want to work with object instances rather than strings.  Here's the markup for building a radio button group from a theoretical array of employee objects:

```markup
<label repeat.for="employee of employees">
  <input type="radio" name="emps" model.bind="employee" checked.bind="$parent.employeeOfTheMonth" />
  ${employee.fullName}
</label>
```

The primary difference between this example and the previous example is we're storing the input values in a special property, `model`, instead of the input element's `value` property which only accepts strings.

You can also bind a radio group to a boolean property like this:

```markup
<label><input type="radio" name="tacos" model.bind="null" checked.bind="likesTacos" />Unanswered</label>
<label><input type="radio" name="tacos" model.bind="true" checked.bind="likesTacos" />Yes</label>
<label><input type="radio" name="tacos" model.bind="false" checked.bind="likesTacos" />No</label>
```

<h4 id="checkboxes"><a href="#checkboxes">checkboxes</a></h4>

To better support multi-select scenarios Aurelia enables binding an input element's checked property to an array.  Here's how you'd bind an array of strings, `favoriteColors`:

```markup
<label repeat.for="color of colors">
  <input type="checkbox" value.bind="color" checked.bind="$parent.favoriteColors" />
  ${color}
</label>
```

This works with arrays of objects as well:

```markup
<label repeat.for="employee of employees">
  <input type="checkbox" model.bind="employee" checked.bind="$parent.favoriteEmployees" />
  ${employee.fullName}
</label>
```

You can of course bind each checkboxes to it's boolean properties like this:

```markup
<li><label><input type="checkbox" checked.bind="wantsFudge" />Fudge</label></li>
<li><label><input type="checkbox" checked.bind="wantsSprinkles" />Sprinkles</label></li>
<li><label><input type="checkbox" checked.bind="wantsCherry" />Cherry</label></li>
```

<h4 id="innerhtml"><a href="#innerhtml">innerHTML</a></h4>

You can bind an element's `innerHTML` property using the `innerhtml` attribute:

``` markup
<div innerhtml.bind="htmlProperty"></div>
<div innerhtml="${htmlProperty}"></div>
```

Aurelia provides a simple html sanitization converter that can be used like this:

``` markup
<div innerhtml.bind="htmlProperty | sanitizeHtml"></div>
<div innerhtml="${htmlProperty | sanitizeHtml}"></div>
```

You're encouraged to use a more complete html sanitizer such as [sanitize-html](https://www.npmjs.com/package/sanitize-html).  Here's how you would build a converter using this package:

``` bash
jspm install npm:sanitize-html
```

``` javascript
import sanitizeHtml from 'sanitize-html';

export class MySanitizeHtmlValueConverter {
  toView(untrustedHtml) {
    return sanitizeHtml(untrustedHtml);
  }
}
```

> NOTE:  Binding using the `innerhtml` attribute simply sets the element's `innerHTML` property.  The markup does not pass through Aurelia's templating system.  Binding expressions and require elements will not be evaluated.  A solution for this scenario is being tracked in [aurelia/templating#35](https://github.com/aurelia/templating/issues/35).

<h4 id="textcontent"><a href="#textcontent">textContent</a></h4>

You can bind an element's `textContent` property using the `textcontent` attribute:

``` markup
<div textcontent.bind="stringProperty"></div>
<div textcontent="${stringProperty}"></div>
```

Two-way data-binding is supported with `contenteditable` elements:

``` markup
<div textcontent.bind="stringProperty" contenteditable="true"></div>
```

<h4 id="style"><a href="#style">style</a></h4>

You can bind a css string or object to an element's `style` attribute:

``` javascript
export class Foo {
  constructor() {
    this.styleString = 'color: red; background-color: blue';

    this.styleObject = {
      color: 'red',
      'background-color': 'blue'
    };
  }
}
```

``` markup
<div style.bind="styleString"></div>
<div style.bind="styleObject"></div>
```

Use the `style` attribute's alias, `css` when doing string interpolation to ensure your application is compatible with Internet Explorer:

``` markup
<!-- good: -->
<div css="width: ${width}px; height: ${height}px;"></div>

<!-- incompatible with Internet Explorer: -->
<div style="width: ${width}px; height: ${height}px;"></div>
```

<h4 id="adaptive-binding"><a href="#adaptive-binding">Adaptive Binding</a></h4>

Aurelia has an adaptive binding system that chooses from a number of strategies when determining how to most efficiently observe changes.  For more info on how this works checkout [this post](http://blog.durandal.io/2015/04/03/aurelia-adaptive-binding/).  For the most part you don't need to think about these details however it does help to be aware of scenarios that lead to inefficient use of the binding system.

**The #1 thing to be aware of is that computed properties (properties with getter functions) are observed using dirty-checking.**  More efficient strategies such as Object.observe and property rewriting are not compatible with these types of properties.

In today's browser environment dirty-checking is a necessary evil.  Very few browsers support Object.observe at the time of this writing.  Aurelia's dirty-checking mechanism is similar to that used in [Polymer](https://www.polymer-project.org/).  It's very efficient and utilizes Aurelia's micro-task-queue to batch updates to the DOM.

A few bindings using dirty-checking will not cause performance problems in your application.  Extensive use of dirty-checking may.  Fortunately there's a way you can avoid dirty-checking simple computed properties.  Consider the 'fullName' property in the example below:

```javascript
export class Person {
  firstName = 'John';
  lastName = 'Doe';

  @computedFrom('firstName', 'lastName')
  get fullName(){
    return `${this.firstName} ${this.lastName}`;
  }
}
```

We've used the `@computedFrom` decorator to provide a hint to the Aurelia binding system.  The binding system now knows to only check `fullName` for changes when `firstName` or `lastName` changes.

It's also important to be mindful of how dirty-checking works.  When a property is "dirty-checked" the binding system periodically checks whether the property's current value matches the previously observed value for the property.  By default this check happens every 120 milliseconds.  This means your property's getter function has the potential to be called quite often which means it should be as efficient as possible.  You should also avoid unnecessarily returning new instances of objects or arrays.  Consider the following view:

```markup
<template>
  <label for="search">Search Issues:</label>
  <input id="search" type="text" value.bind="searchText" />
  <ul>
    <li repeat.for="issue of filteredIssues">${issue.abstract}</li>
  </ul>
</template>
```

Naive view model implementation:

```javascript
export class IssueSearch {
  searchText = '';

  constructor(allIssues) {
    this.allIssues = allIssues;
  }

  // this returns a new array instance on every call which will in-turn result in a lot of DOM updates.
  get filteredIssues() {
    if (this.searchText === '')
      return [];
    return this.allIssues.filter(x => x.abstract.indexOf(this.searchText) !== -1);
  }
}
```

Improved view model implementation:

```javascript
export class IssueSearch {
  filteredIssues = [];
  _searchText = '';

  constructor(allIssues) {
    this.allIssues = allIssues;
  }

  get searchText() {
    return this._searchText;
  }
  set searchText(newValue) {
    this._searchText = newValue;
    if (newValue === '') {
      this.filteredIssues = [];
    } else {
      this.filteredIssues = this.allIssues.filter(x => x.abstract.indexOf(this.searchText) !== -1);
    }
  }
}
```

<h3 id="html-extensions"><a href="#html-extensions">HTML Extensions</a></h3>

In addition to databinding, you also have the power of Aurelia's HTML extensions. There are two types:

* Custom Elements - Extend HTML with new tags! Your custom elements can have their own views (which use databinding and other html extensions) and optionally leverage [ShadowDOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/) (even if the browser doesn't support it).
* Custom Attributes - Extend HTML with new attributes which can be added to existing or custom elements. These attributes add new behavior to the elements.

Naturally, all of this works seamlessly with databinding. Let's look at the set of Custom Elements and Attributes that Aurelia provides for you and which are available globally in every view.

<h4 id="show"><a href="#show">show</a></h4>

The `show` Custom Attribute allows you to conditionally display an HTML element. If the value of show is `true` the element will be displayed, otherwise it will be hidden. This attribute does not add/remove the element from the DOM, but only changes its visibility. Here's an example:

```markup
<div show.bind="isSaving" class="spinner"></div>
```

When the `isSaving` property is true, the `div` will be visible, otherwise it will be hidden.

<h4 id="if"><a href="#if">if</a></h4>

The `if` Custom Attribute allows you to conditionally add/remove an HTML element. If the value is true, the element will also be present in the DOM, otherwise it will not.

```markup
<div if.bind="isSaving" class="spinner"></div>
```

This example looks similar to that of `show` above. The difference is that if the binding expression evaluates to false, the `div` will be removed from the DOM, rather than just hidden.

If you need to conditionally add/remove a group of elements and you cannot place the `if` attribute on a parent element, then you can wrap those elements in a template tag which has the `if` attribute. Here's what that would look like:

```markup
<template if.bind="hasErrors">
    <i class="icon error"></i>
    ${errorMessage}
</template>
```

>**Note:** It's important to note that you should NOT add an `if` behavior around a `<content>` element. The ShadowDOM does not support dynamically adding these elements they way you might expect. Instead, use a `show` behavior on a parent element.

<h4 id="repeat"><a href="#repeat">repeat</a></h4>

The `repeat` Custom Attribute allows you to render a template multiple times, once for each item in an array. Here's an example of how that renders out a list of customer names:

```markup
<ul>
    <li repeat.for="customer of customers">${customer.fullName}</li>
</ul>
```

An important note about the repeat attribute is that it works in conjunction with the `.for` binding command. This binding command interprets a special syntax in the form "item of collection" where "item" is the local name you will use in the template and "collection" is a normal binding expression that evaluates to an array or map.

Speaking of Maps, here's how you would bind to an ES6 Map:

```markup
<ul>
  <li repeat.for="[id, customer] of customers">${id} ${customer.fullName}</li>
</ul>
```

If instead of iterating over a collection you would rather iterate a specified number of times, you can instead use the syntax "i of count" where "i" is the index of the iteration and "count" is a binding expression that evaluates to an integer.

```markup
<ul>
  <li repeat.for="i of rating">*</li>
</ul>
```

> **Note:**: Like the `if` attribute, you can also use a `template` tag to group a collection of elements that don't have a parent element.

Each item that is being repeated by the `repeat` attribute has several special contextual values available for binding:

* `$parent` - At present, the main view model's properties and methods are not visible from within the repeated item. We hope to remedy this in an update soon. For the mean time, you can access that view-model with `$parent`.
* `$index` - The index of the item in the array.
* `$first` - True if the item is the first item in the array.
* `$last` - True if the item is the last item in the array.
* `$even` - True if the item has an even numbered index.
* `$odd` - True if the item has an odd numbered index.

<h4 id="compose"><a href="#compose">compose</a></h4>

The `compose` Custom Element enables you to dynamically render UI into the DOM. Imagine you have a heterogeneous array of items, but each has a type property which tells you what it is. You can then do something like this:

```markup
<template repeat.for="item of items">
    <compose
      model.bind="item"
      view-model="widgets/${item.type}">
    </compose>
</template>
```

Now, depending on the _type_ of the item, the `compose` element will load a different view-model (and view) and render it into the DOM. If the view-model has an `activate` method, the `compose` element will call it and pass in the `model` as a parameter. The `activate` method can even return a `Promise` to cause the composition process to wait until after some async work is done before actually databinding and rendering into the DOM.

The `compose` element also has a `view` attribute which can be used in the same way as `view-model` if you don't wish to leverage the standard view/view-model convention. If you specify a `view` but no `view-model` then the view will be databound to the surrounding context.

```markup
<template repeat.for="item of items">
    <compose view="my-view.html"></compose>
</template>
```

If you would like to databind a particular object when using only `view` rather than a full fledged view model (perhaps as part of a repeat), you may do so by binding the `view-model` directly. Now you will be able to use properties of that object directly in your `view`:

```markup
<template>
    <div repeat.for="item of items">
      <compose view="my-view.html" view-model.bind="item">
    </div>
</template>
```

What if you want to determine the view dynamically based on data though? or runtime conditions? You can do that too by implementing a `getViewStrategy()` method on your view-model. It can return a relative path to the view or an instance of a `ViewStrategy` for custom view loading behavior. The nice part is that this method is executed after the `activate` callback, so you have access to the model data when determining the view.

<h4 id="global-behavior"><a href="#global-behavior">global-behavior</a></h4>

This is not an HTML enhancement that you will use directly. Rather, it works in conjunction with a custom binding command to dynamically enable the use of jQuery plugins and similar APIs declaratively in HTML. Let's look at an example in order to help clarify the idea:

```markup
<div jquery.modal="show: true; keyboard.bind: allowKeyboard">...</div>
```

This sample is based on the [Bootstrap modal widget](http://getbootstrap.com/javascript/#modals). In this case, the `modal` jQuery widget will be attached to the `div` and it will be configured with its `show` option set to `true` and its `keyboard` option set to the value of the `allowKeyboard` property on the view-model. When the containing view is unbound, the jQuery widget will be destroyed.

This capability combines the special `global-behavior` with custom syntax to enable these dynamic capabilities. The syntax you see here is based on the syntax of the native `style` attribute which lists properties and values separated in the same fashion as above. Note that you can use binding commands such as `.bind` to pass data from your view-model directly to the plugin or `.call` to pass a callback function directly to the plugin.

Here's how it works:

When the binding system sees a binding command that it doesn't recognize, it dynamically interprets it. The attribute name is mapped to a global binding handler which interprets the binding command. The handler can use the values to create an options object which it can pass to the plugin. When the view is unbound, the handler can also cleanup after itself. In this case the jQuery handler knows the pattern for instantiating plugins and using the `destroy` method to cleanup.

> **Note:** The `global-behavior` has a handlers list you must configure. It is only configured with jQuery by default. You can turn all of this off, if you desire, but it makes it easy to take advantage of basic jQuery plugins without any work on your part.

<h2 id="routing"><a href="#routing">Routing</a></h2>

There are many different application styles you could be called upon to create. From navigation apps, to dashboards, to MDI interfaces, Aurelia can handle them all. In many of these cases a key component of your architecture is a client-side router, capable of translating url changes into application state.

If you've read the getting started guide, you know that there are two parts to routing. First, there's the `Router` which lives in your view-model. It's configured with route information and controls navigation. Then, there's the `router-view` which lives in the view and is responsible for displaying whatever the router identifies as the current state.

Let's look at an example configuration.

```javascript
export class App {
  configureRouter(config, router){
    this.router = router;

    config.title = 'Aurelia';
    config.map([
      { route: ['', 'home'],       name: 'home',       moduleId: 'home/index' },
      { route: 'users',            name: 'users',      moduleId: 'users/index',   nav: true },
      { route: 'users/:id/detail', name: 'userDetail', moduleId: 'users/detail' },
      { route: 'files*path',       name: 'files',      moduleId: 'files/index',   href:'#files',   nav: true }
    ]);
  }
}
```

We begin by implementing the `configureRouter` method. We can optionally set a `title` property to be used in constructing the document's title, but the most important part is setting up the routes. The router's `map` method takes a simple JSON data structure representing your route table. The two most important properties are `route` (a string or array of strings), which defines the route pattern, and `moduleId`, which has the *relative* module Id path to your view-model. You can also set a `name` property, to be used to generate a link to the route later, a `title` property, to be used when generating the document's title, a `nav` property indicating whether or not the route should be included in the navigation model (it can also be a number indicating order) and an `href` property which you can use to bind to in the _navigation model_.

So, what options do you have for the route pattern?

* static routes
    - ie 'home' - Matches the string exactly.
* parameterized routes
    - ie  'users/:id/detail' - Matches the string and then parses an `id` parameter. Your view-model's `activate` callback will be called with an object that has an `id` property set to the value that was extracted from the url.
* wildcard routes
    - ie 'files*path' - Matches the string and then anything that follows it. Your view-model's `activate` callback will be called with an object that has a `path` property set to the wildcard's value.

All routes with a truthy `nav` property are assembled into a `navigation` array. This makes it really easy to use databinding to generate a menu structure. Another important property for binding is the `isNavigating` property. Here's some simple markup that shows what you might pair with the view-model shown above:

```markup
<template>
  <ul>
    <li class="loader" if.bind="router.isNavigating">
      <i class="fa fa-spinner fa-spin fa-2x"></i>
    </li>
    <li repeat.for="item of router.navigation">
      <a href.bind="item.href">${item.title}</a>
    </li>
  </ul>

  <router-view></router-view>
</template>
```

<h3 id="the-screen-activation-lifecycle"><a href="#the-screen-activation-lifecycle">The Screen Activation Lifecycle</a></h3>

Whenever the router processes a navigation, it enforces a strict lifecycle on the view-models that it is navigating to and from. There are four stages in the lifecycle. You can opt-in to any of them by implementing the appropriate method on your view-model's class. Here's a list of the lifecycle callbacks:

* `canActivate(params, routeConfig, navigationInstruction)` - Implement this hook if you want to control whether or not your view-model _can be navigated to_. Return a boolean value, a promise for a boolean value, or a navigation command.
* `activate(params, routeConfig, navigationInstruction)` - Implement this hook if you want to perform custom logic just before your view-model is displayed. You can optionally return a promise to tell the router to wait to bind and attach the view until after you finish your work.
* `canDeactivate()` - Implement this hook if you want to control whether or not the router _can navigate away_ from your view-model when moving to a new route. Return a boolean value, a promise for a boolean value, or a navigation command.
* `deactivate()` - Implement this hook if you want to perform custom logic when your view-model is being navigated away from. You can optionally return a promise to tell the router to wait until after your finish your work.

The `params` object will have a property for each parameter of the route that was parsed, as well as a property for each query string value. `routeConfig` will be the original route configuration object that you set up. `routeConfig` will also have a new `navModel` property, which can be used to change the document title for data loaded by your view-model. For example:

```javascript
activate(params, routeConfig) {
  this.userService.getUser(params.id)
    .then(user => {
      routeConfig.navModel.setTitle(user.name);
    });
}
```

> **Note:** A _Navigation Command_ is any object with a `navigate(router)` method. When one is encountered, the navigation will be cancelled and control will be passed to the navigation command. One navigation command is provided out of the box: `Redirect`.

<h3 id="child-routers"><a href="#child-routers">Child Routers</a></h3>

If you haven't read the "Get Started" guide, we recommend that you do that now and pay special attention to the section titled "Bonus: Leveraging Child Routers".

Whenever you set up a route to map to a view-model, that view-model can actually contain its own router...and when you set up routes with that...those view-models can have their own routers...and so on. The route patterns are relative to the parent router and the module and view ids are relative to the view-model itself. This allows you to easily encapsulate features or child applications as well as handle complex hierarchical state.

A child router is just a router like any other. So, everything we've discussed above applies. To add a child router, just implement the `configureRouter` method again. The screen activation lifecycle discussed above applies to child routers as well. Each phase of the lifecycle is run against the entire router hierarchy before moving on to the next phase. The activate hooks run from top to bottom and the deactivate hooks run from bottom to top.

<h3 id="conventional-routing"><a href="#conventional-routing">Conventional Routing</a></h3>

As with everything in Aurelia, we have strong support for conventions. So, you can actually choose to dynamically route rather than pre-configuring all your routes up front. Here's how you configure a router to do that:

```javascript
export class App {
  configureRouter(config){
    config.mapUnknownRoutes(instruction => {
      //check instruction.fragment
      //set instruction.config.moduleId
    });
  }
}
```

All you have to do is set the `instruction.config.moduleId` property and you are good to go. You can also return a promise from `mapUnknownRoutes` in order to asynchronously determine the destination.

>**Note:** Though not necessarily related to conventional routing, you may sometimes have a need to asynchronously configure your router. For example, you may need to call a web service to get user permissions before setting up routes. To do this, return a promise from `configureRouter`.

<h3 id="customizing-the-navigation-pipeline"><a href="#customizing-the-navigation-pipeline">Customizing the Navigation Pipeline</a></h3>

The router pipeline is composed out of separate steps that run in succession. Each of these steps has the ability to modify what happens during routing, or stop the routing altogether. The pipeline also contains a few extensibility points where you can add your own steps. These are `authorize` and `modelbind`. `authorize` happens before `modelbind`. These extensions are called route filters.

The sample below shows how you can add authorization to your application:

```javascript
import {Redirect} from 'aurelia-router';

export class App {
  configureRouter(config) {
    config.title = 'Aurelia';
    config.addPipelineStep('authorize', AuthorizeStep); // Add a route filter to the authorize extensibility point.
    config.map([
      { route: ['welcome'],    name: 'welcome',       moduleId: 'welcome',      nav: true, title:'Welcome' },
      { route: 'flickr',       name: 'flickr',        moduleId: 'flickr',       nav: true, auth: true },
      { route: 'child-router', name: 'childRouter',   moduleId: 'child-router', nav: true, title:'Child Router' },
      { route: '', redirect: 'welcome' }
    ]);
  }
}

class AuthorizeStep {
  run(routingContext, next) {
    // Check if the route has an "auth" key
    // The reason for using `nextInstructions` is because
    // this includes child routes.
    if (routingContext.nextInstructions.some(i => i.config.auth)) {
      var isLoggedIn = /* insert magic here */false;
      if (!isLoggedIn) {
        return next.cancel(new Redirect('login'));
      }
    }

    return next();
  }
}
```

These extensibility points are in and of themselves small pipelines, and multiple steps can be added to each of them. For instance, if in addition to the `AuthorizeStep` above (which would just check that a user is logged in), you could add an `IsAdminStep` to the `authorize` extensibility point. They would then run in succession.

It's also possible to create your own named filters by simply passing a different name into `addPipelineStep`. This can be used like in the example below:

```javascript
config.addPipelineStep('myname', MyFirstStep); // Transparently creates the pipeline "myname" if it doesn't already exist.
config.addPipelineStep('myname', MySecondStep); // Adds another step to it.
config.addPipelineStep('modelbind', 'myname'); // Makes the entire `myname` pipeline run as part of the `modelbind` pipeline.
```

<h3 id="configuring-push-state"><a href="#configuring-push-state">Configuring PushState</a></h3>

If you'd prefer to get rid of the `#` (hashes) in your URLs, then you're going to have to enable `pushState` in your app. Good thing Aurelia supports that! You will also have to do some work on the server side to ensure it works properly. Let's start with the Aurelia side of the equation.

First you need to tell Aurelia in the `router` `config` that you want to use `pushState` like so:

``` javascript
export class App {
  configureRouter(config) {
    config.title = 'Aurelia';
    config.options.pushState = true; // <-- this is all you need here
    config.map([
      { route: ['welcome'],    name: 'welcome',     moduleId: 'welcome',      nav: true, title:'Welcome' },
      { route: 'flickr',       name: 'flickr',      moduleId: 'flickr',       nav: true, auth: true },
      { route: 'child-router', name: 'childRouter', moduleId: 'child-router', nav: true, title:'Child Router' },
      { route: '',             redirect: 'welcome' }
    ]);
  }
}
```

Add [a base tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base) to the head of your html document. If you're using JSPM, you will also need to configure it with a `baseURL` corresponding to your base tag's `href`.

```js
System.config({
  "baseURL": "/",
  ...
```

Next, the server side needs to be configured to send back the same `index.html` file regardless of the request being made because all the routing is done client side. So, if you're using the `gulp watch` task with `browsersync` as per the navigation sample, then you can modify your setup like so:

From the console in the root of your project, run the following:

```shell
npm install --save connect-history-api-fallback
```

This will download and install the middleware plugin you need for this. Then open up your _build/tasks_ folder and locate the _serve_ task. Open that and put this somewhere near the top with the other require statements:

``` javascript
var historyApiFallback = require('connect-history-api-fallback')
```

Lower down you can modify the `serve` task to use the new `middleware`:

``` javascript
gulp.task('serve', ['build'], function(done) {
  browserSync({
    open: false,
    port: 9000,
    server: {
      baseDir: ['.'],
      middleware: [historyApiFallback(), function (req, res, next) { // it's the first one in the array
        res.setHeader('Access-Control-Allow-Origin', '*');
        next();
      }]
    }
  }, done);
});
```

Now your node server should behave itself and let Aurelia deal with the routing.

If you're using a .NET server side framework such as ASP.NET MVC then config is as follows:

* Create a Controller and call it ApplicationController or what ever you want to call it. It should look something like this:

```csharp
public class ApplicationController : Controller {
  public ActionResult Index() {
    return View();
  }
}
```

* Create an "index.cshtml" view in your Views folder.

* Setup your routing configuration like this:

```csharp
context.MapRoute(
  name: "AureliaRouting",
  url: "{*.}",
  defaults: new { controller = "Application", action = "Index" }
);
```
Note that with the above you will be forced to use a Razor view file. If you want to use a regular HTML file, there are different ways to do it. [This SO article might help you](http://stackoverflow.com/questions/20871938/render-html-file-in-asp-net-mvc-view).

If you are using [Nancy FX](http://nancyfx.org), then the config is just as simple. Locate your `IndexModule.cs` or whatever you called it and make sure it looks something like this and all will be well:

``` csharp
public class IndexModule : NancyModule {
  public IndexModule()     {
    this.Get["/robots.txt"] = p => this.Response.AsFile("robots.txt");
    this.Get["/sitemap.xml"] = p => this.Response.AsFile("sitemap.xml");
    this.Get["/"] = x => this.View["index"];
    this.Get["/{path*}"] = x => this.View["index"];
  }
}
```

Similar techniques can be used in other server environments - you just need to make sure that whatever server you're using, it needs to send back the same `index.html` regardless of the request being made. All server side frameworks should be able to achieve this. Aurelia will figure out which page to load based on its own route data.

<h3 id="reusing-an-existing-vm"><a href="#reusing-an-existing-vm">Reusing an existing VM</a></h3>

Sometimes you might want to use the same VM for multiple routes. By default Aurelia will see those routes as aliases to the same VM and thus only perform the build and attach process as well as the complete life-cycle once. This might not be exactly what you are looking for. Take the following router example:

```javascript
export class App {
  configureRouter(config) {
    config.title = 'Aurelia';
    config.map([
      { route: 'product/a',    moduleId: 'product',     nav: true },
      { route: 'product/b',    moduleId: 'product',     nav: true },
    ]);
  }
}
```

Since the VM's life-cycle is called only once you may have problems to recognize that the user switched the route from `Product A` to `Product B`.

To work around this issue implement the method `determineActivationStrategy` in your VM and return hints for the router about what you'd like to happen. E.g in order to force a rebuild of the VM implement it like this:

```javascript
import {activationStrategy} from 'aurelia-router';

export class YourViewModel {
  determineActivationStrategy(){
    return activationStrategy.replace;
  }
}
```

> **Note:** Additionally, you can add an `activationStrategy` property to your route config if the strategy is always the same and you don't want that to be in your view-model code.

If you just want to force a refresh of the life-cycle (useful with `<compose>` bindings) you may do something like the following:

```javascript
import {activationStrategy} from 'aurelia-router';

export class YourViewModel {
  determineActivationStrategy(){
    return activationStrategy.invokeLifecycle;
  }
}
```

> **Note:** Keep in mind that by forcing refreshes, Aurelia has to rebuild the complete VM. As for performance reasons a simple observer on the `router.currentInstruction` might be sufficient for scenarios where you'd simply like to exchange some data.

<h3 id="rendering-multiple-view-ports"><a href="#rendering-multiple-view-ports">Rendering multiple ViewPorts</a></h3>
Sometimes you need to render content in more than one area of the page. Aurelia's router lets you specify multiple `router-view`s to be activated by a single route. First, add named `router-view` elements to your view:

```markup
<template>
  <div class="page-host">
    <router-view name="left"></router-view>
  </div>
  <div class="page-host">
    <router-view name="right"></router-view>
  </div>
</template>
```

Then in your route config, specify which modules should be activated for each named `router-view`.

```javascript
configureRouter(config){
  config.map([{
    route: 'edit',
      viewPorts: {
        left: {
          moduleId: 'editor'
        },
        right: {
          moduleId: 'preview'
        }
      }
    }]);
}
```

If you don't name a `router-view`, it will be available under the name `'default'`.

<h3 id="generating-route-urls"><a href="#generating-route-urls">Generating route URLs</a></h3>

If you need a to create a URL that matches an existing route, the router can generate one for you.

```javascript
router.generate('userDetail', { id: 123 });
```

The first parameter is the route name, as specified in the route config. The second parameter is an object with route parameters to fill in to the route template. Any properties of the object that don't map to route parameters will be automatically appended to the query string.

If you want to navigate to the generated URL, use `router.navigateToRoute('userDetail', { id: 123 })`.

If you simply want to render an anchor in your view, you can use the `route-href` custom attribute.

```markup
<a route-href="route: userDetail; params.bind: { id: user.id }">${user.name}</a>
```

<h2 id="extending-html"><a href="#extending-html">Extending HTML</a></h2>

Aurelia has a powerful and extensible HTML template compiler. The compiler itself is just an algorithm for interacting with these extensions. Out of the box, Aurelia provides two extensions: _Custom Elements_ and _Custom Attributes_.

These extensions are not visible to the compiler by default. There are three main ways to plug them in:

* Use the `require` element to require an extension in a view. The `from` attribute specifies the relative path to the extension's module. The extension will be locally defined.
* Use the Aurelia object during your bootstrapping phase to call `.globalizeResources(...resourcePaths)` to register extensions with global visibility in your application.
* Install a plugin that registers extensions with global visibility in your application.

>**Note:** A recommended practice for your own apps is to place all your app-specific extensions, value converters, etc. into a _resources_ folder. Then create an _index.js_ file that turns them all into an internal feature plugin. Finally, install the feature during your app's bootstrapping phase using `aurelia.use.feature('resources')`. This will keep your resources located in a known location, along with their registration code. It will also keep your configuration file clean and simple.

All extensions can opt into the view lifecycle by implementing any of the following hooks:

* `created(view:View)` - Invoked after both the view and view-model have been created. Allows your behavior to have direct access to the View instance.
* `bind(bindingContext:any)` - Invoked when the databinding engine binds the view. The binding context is the instance that the view is databound to.
* `unbind()` - Invoked when the databinding engine unbinds the view.
* `attached()` - Invoked when the view that contains the extension is attached to the DOM.
* `detached()` - Invoked when the view that contains the extension is detached from the DOM.

>**Note:** If you choose to implement the `bind` callback, the initial binding of your extension will flow a little differently. Usually, if you have callbacks for your extension's bindable properties, these are each individually called during the bind phase. However, if you add the `bind` callback, they will not be called during initialization. Rather, the `bind` callback will be called once all properties have their initial bound values set. This is an important and useful characteristic, particularly for complex extensions which may not want to "act" until they have all evaluated values available.

<h3 id="custom-attributes"><a href="#custom-attributes">Custom Attributes</a></h3>

Custom Attributes extend HTML with functionality by adding new behavior to existing HTML elements. Common uses for Custom Attributes include:

* Wrapping jQuery and similar plugins (when the `global-behavior` is insufficient).
* Shortcuts for common style, class or attribute bindings.
* Just about anything that needs to change an existing HTML element or even a Custom Element which you cannot directly alter.

Custom Attributes tend to represent cross-cutting concerns. For example you might create a custom tooltip attribute that you can then attach to any element. This is a better idea than building tooltip functionality directly into every custom element you create.

Let's look at one of Aurelia's own Custom Attribute implementations: `show`. Here's how it is used:

```markup
<div show.bind="isSaving" class="spinner"></div>
```

The `show` attribute will conditionally apply a css class to an element based on the falseness of its value. (The css class, when applied, hides the element.) Here's the implementation:

```javascript
import {inject, customAttribute} from 'aurelia-framework';

@customAttribute('show')
@inject(Element)
export class Show {
  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue){
    if (newValue) {
      this.element.classList.remove('aurelia-hide');
    } else {
      this.element.classList.add('aurelia-hide');
    }
  }
}
```

The first thing to note is that Custom Attributes are classes and follow the same patterns we've already seen. Notice that the decorators play an important role in defining the attribute. Here's what they are doing:

* `@customAttribute('show')` - Indicates that this class is a custom attribute so the HTML compiler knows how this class "plugs in". It will be recognized by the compiler any time it sees an attribute named "show".
* `inject` - This is part of the dependency injection system; the same as you've seen before. Custom Attributes can have the Element they are placed on injected into the constructor. That is what is happening here. All you have to do use use the browser's `Element` type to indicate that.

There are a few other interesting things that happen too.

* Attributes in html have a value. So, your Custom Attribute class will have a `value` property that is kept in sync with the HTML. If you implement a `valueChanged` method, it will be invoked any time the attribute's value changes. The first argument will be the new value and the second will be the old value.

What about conventions?

If your class's export name matches the pattern {SomeName}CustomAttribute, then you don't need to include the `@customAttribute` decorator at all. The attribute name will be inferred from the export name by stripping off "CustomAttribute" and lowercasing and hyphenating the remaining part of the name. ie. some-name

These conventions mean that we can actually define our `show` attribute like this:

```javascript
export class ShowCustomAttribute {
  static inject = [Element]; //showing non-decorator method here for variety

  constructor(element) {
    this.element = element;
  }

  valueChanged(newValue){
    if (newValue) {
      this.element.classList.remove('aurelia-hide');
    } else {
      this.element.classList.add('aurelia-hide');
    }
  }
}
```

> **Note:** So, why doesn't Aurelia itself leverage these conventions internally? Any time you are creating a 3rd party library, it's best to be explicit. You don't know whether or not developers consuming your library will have changed Aurelia's conventions, thus breaking your library. In order to prevent this, always be explicit by using decorators. Inside your own apps though, you can use the conventions all you want to simplify development.

<h4 id="options-attributes"><a href="#options-properties">Options Attributes</a></h4>

You may be wondering what to do if you want to create a Custom Attribute with multiple properties, since attributes usually map to a single value. It's actually quite simple. Just create several `bindable` properties:

```javascript
import {customAttribute, bindable} from 'aurelia-framework';

@customAttribute('my-attribute')
export class MyAttribute {
  @bindable foo;
  @bindable bar;
}
```

This creates a Custom Attribute named `my-attribute` with two properties `foo` and `bar`. Each of these properties are available directly on the class. Each can have optional change callbacks, `fooChanged` and `barChanged` respectively. However, they are configured in HTML a bit different. Here's how that would be done:

```markup
<div my-attribute="foo: some literal value; bar.bind: some.expression"></div>
```

Notice that we don't use a binding command on the attribute itself. Instead, we can use them on each individual property inside the attribute's value. You can use literals as well as the standard binding commands.

> **Note:** You don't use `delegate` or `trigger` commands inside an options attribute. Those are always attached to the element itself, since they work directly with native DOM events. However, you can use `call`.

If you aren't using ES2016 property initializers, you can put the `@bindable` decorator directly on the class. Just be sure to provide the property name like this `@bindable('propertyName')`. To specify more details for a bindable property, you should pass an options object instead like this:

```javascript
@bindable({
  name:'myProperty', //name of the property on the class
  attribute:'my-property', //name of the attribute in HTML
  changeHandler:'myPropertyChanged', //name of the method to invoke when the property changes
  defaultBindingMode: bindingMode.oneWay, //default binding mode used with the .bind command
  defaultValue: undefined //default value of the property, if not bound or set in HTML
})
```

The defaults and conventions are shown above. So, you would only need to specify these options if you need to deviate.

> **Note:** There is also a special `@dynamicOptions` decorator. This allows a custom attribute to have a dynamic set of properties which are all mapped from the options attribute syntax into the class at runtime. Don't declare `bindable` properties. Simply add a single `@dynamicOptions` decorator and anything the consumer lists in the options attribute syntax will be mapped. To be notified on when these dynamic properties change, implement a method with the following signature on your class: `propertyChanged(propertyName, newValue, oldValue)`. Actually, you can implement this on any behavior.

> **Note:** Remember that all decorators are available on the `Decorators` helper and can be specified with a static `decorators` property or method if you prefer (or if you are using a language that doesn't support decorators). See the CoffeeScript examples above for details.

<h4 id="template-controllers"><a href="#template-controllers">Template Controllers</a></h3>

Custom Attributes can indicate that they are a Template Controller with the `@templateController` decorator. This indicates that they convert DOM into an inert HTML template. The custom attribute class can then decide when and where (or how many times) to instantiate the template in the DOM. Examples of this are the `if` and `repeat` attributes. Simply place one of these on a DOM node and it becomes a template, controlled by the Custom Attribute class.

Let's take a look at the implementation of the `if` Custom Attribute to see how one of these is put together. Here's the full source code:

```javascript
import {BoundViewFactory, ViewSlot, customAttribute, templateController, inject} from 'aurelia-framework';

@customAttribute('if')
@templateController
@inject(BoundViewFactory, ViewSlot)
export class If {
  constructor(viewFactory, viewSlot){
    this.viewFactory = viewFactory;
    this.viewSlot = viewSlot;
    this.showing = false;
  }

  valueChanged(newValue){
    if (!newValue) {
      if(this.view){
        this.viewSlot.remove(this.view);
        this.view.unbind();
      }

      this.showing = false;
      return;
    }

    if(!this.view){
      this.view = this.viewFactory.create();
    }

    if (!this.showing) {
      this.showing = true;

      if(!this.view.bound){
        this.view.bind();
      }

      this.viewSlot.add(this.view);
    }
  }
}
```

Before we dig into the unique aspects, let me remind you of what you see here that is similar. First, we have a simple class with decorators. It also has a single `value` property by default which can be observed by adding a `valueChanged` callback.

Ok, what's different? Take a look at the constructor. We have two unique items being injected: `BoundViewFactory` and `ViewSlot`.

The `BoundViewFactory` is capable of generating instances of the the HTML template that the attribute is attached to. No need to worry about compiling, etc. That's taken care of for you. Why is it called "Bound" View Factory though? Well, it's already referencing the parent binding context. It's "bound" in a sense. So, if you call its `create` method it will instantiate a new View from the template which will be bound to that context. This is what you want with an `if` attribute. It's not what you want with a `repeat` attribute. In that case, each time you call `create` you want a view bound to a particular array item. To achieve this, simply pass any object you want the view to be bound against into the `create` method.

The `ViewSlot` represents the slot or location within the DOM that the template was extracted from. This is usually the location that you want to add View instances to.

>**Note**: Unlike previous attributes, a template controller works more directly with the _primitives_ of the framework. Views, ViewFactories and ViewSlots are all low level parts of the templating engine.

Take a close look at the `valueChanged` callback. Here you can see where the `if` attribute is creating the view and adding it to the slot, based on the truthiness of the value. There are a few important details of this:

* The attribute always calls `bind` on the View _before_ adding it to the ViewSlot. This ensures that all internal bindings are initially evaluated outside of the live DOM. This is important for performance.
* Similarly, always call `unbind` _after_ removing the View from the DOM.
* After the View is initially created, the `if` attribute does not throw it away even when the value becomes false. It caches the instance. Aurelia can re-use Views and even re-target them at different binding contexts. Again, this is important for performance, since it eliminates needless re-creation of Views.


<h3 id="custom-elements"><a href="#custom-elements">Custom Elements</a></h3>

Custom Elements add new tags to your HTML markup. Each Custom Element can have its own view template which can be rendered into the Light DOM or the Shadow DOM. Custom Elements can also have any number of properties which they surface as attributes in HTML for databinding support and which they can databind to inside their view template.

Why don't we create a simple custom element so that we can see how that works? We'll make an element that says hello to someone, called `say-hello`. Here's how we want to be able to use it when we're done:

```markup
<template>
    <require from="say-hello"></require>

    <input type="text" ref="name">
    <say-hello to.bind="name.value"></say-hello>
</template>
```

So, how do we build this? Well, we're going to start with a class, just like we did with the Custom Attribute. Here's what it looks like:

#### say-hello.js
```javascript
import {customElement, bindable} from 'aurelia-framework';

@customElement('say-hello')
export class SayHello {
  @bindable to;

  speak(){
    alert(`Hello ${this.to}!`);
  }
}
```

If you read the section on Custom Attributes, then you know what this does. There's some conventions too, which means we can do this if we want:

#### say-hello.js (with conventions)
```javascript
import {bindable} from 'aurelia-framework';

export class SayHelloCustomElement {
  @bindable to;

  speak(){
    alert(`Hello ${this.to}!`);
  }
}
```

Be default, Custom Elements have a view. Here's the view for ours:

#### say-hello.html
```markup
<template>
    <button click.trigger="speak()">Say Hello To ${to}</button>
</template>
```

As you can see, we've got access to our class's properties and methods. It's important to note that you don't need to declare `@bindable` properties for every property you want to bind to in your template. You only need to declare it for properties you want to exist as attributes on your custom element.

That's really all there is to it. You follow the same view-model/view naming conventions and all the same patterns for custom elements. There are a few unique decorators for custom elements you may also need:

* `@sync(selector)` - Decorates a property to create an array on your class that has its items automatically synchronized based on a query selector against its view.
*  `@processContent(false|Function)` - Tells the compiler that the element's content requires special processing. If you provide `false` to the decorator, the the compiler will not process the content of your custom element. It is expected that you will do custom processing yourself. But, you can also supply a custom function that lets you process the content during the view's compilation. That function can then return true/false to indicate whether or not the compiler should also process the content. The function takes the following form `function(compiler, resources, node, instruction):boolean`
*  `@useView(path)` - Specifies a different view to use.
*  `@noView()` - Indicates that this custom element does not have a view and that the author intends for the element to handle its own rendering internally.
* `@inlineView(markup, dependencies?)` - Allows the developer to provide a string that will be compiled into the view.
* `@containerless()` - Causes the element's view to be rendered without the custom element container wrapping it. This cannot be used in conjunction with `@sync` or `@useShadowDOM`. It also cannot be uses with surrogate behaviors.
* `@useShadowDOM()` - Causes the view to be rendered in the ShadowDOM. When an element is rendered to ShadowDOM, a special `DOMBoundary` instance can optionally be injected into the constructor. This represents the shadow root.

<h3 id="template-parts"><a href="#template-parts">Template Parts</a></h3>

Template part replacement in custom elements allows a custom element to specify certain parts of its view which can be replaced with alternate markup at runtime on a per-instance basis.

If you are using a custom element you can mark any part of it’s view as `replaceable`. Then the consumer of your element can specify a template in the element's content indicating the part they want it to replace in the element’s view.  Use `part="someName"` to identify a part of the template that is replaceable. If it’s not a template for a template controller (repeat or if) then you also need the `replaceable` attribute on the part. Finally, when the consumer wants to replace that part, they add `replace-part="someName"` on a template inside the elements' content to provide the alternate version.

Here's an example that shows how to make the template inside of a repeater replaceable without affecting the `li` container. It also shows how to create the custom element so that the runtime binding context where the custom element is used can be reached by the replaced template.

#### example.js
```javascript
export class Example {
  constructor(){
    this.items = [1,2,3,4,5];
  }

  bind(context){
    this.$parent = context;
  }
}
```

#### example.html
```markup
<template>
  <ul>
    <li class="foo" repeat.for="item of items">
      <template replaceable part="item-template">
        Original: ${item}
      </template>
    </li>
  <ul>
</template>
```

#### welcome.js
```javascript
export class Welcome{
  heading = 'Welcome to the Aurelia Navigation App!';
  firstName = 'John';
  lastName = 'Doe';

  get fullName(){
    return `${this.firstName} ${this.lastName}`;
  }

  welcome(){
    alert(`Welcome, ${this.fullName}!`);
  }
}
```

#### welcome.html
```markup
<template>
  <require from="example"></require>

  <example>
    <template replace-part="item-template">
      Replacement: ${item} ${$parent.$parent.fullName} <button click.delegate="$parent.$parent.welcome()">Test</button>
    </template>
  </example>
</template>
```

<h3 id="surrogate-behaviors"><a href="#surrogate-behaviors">Surrogate Behaviors</a></h3>

Imagine that you want to create a new custom element called `progress-bar`. Naturally, you want your element to be accessible so you want to include all the aria attributes that make sense directly on the custom element. The problem is that you either need to ask the consumer to add those when they use your element, or you need to add them all in code and wire their change events up manually in order to add them on the element. It's not pretty.

Enter Surrogate Behaviors...

Surrogate Behaviors allow you to place bindings and custom attributes directly on the `template` element of the custom element itself. When the element is instantiated, the bindings and behaviors will be attached to the custom element host. In this way the `template` element of your view acts as a surrogate or stand-in for the actual HTML element at runtime. The bindings and behaviors will be bound to the custom element's class. Here's an example of a progress-bar view:

```markup
<template role="progress-bar" aria-valuenow.bind="progress" aria-valuemin="0" aria-valuemax="100">
  <div class="bar">
    <div class="progress" css="width:${progress}%"></div>
  </div>
</template>
```

<h2 id="eventing"><a href="#eventing">Eventing</a></h2>

Eventing is a powerful tool when you need decoupled components of your application to talk to one another. Aurelia supports both standard DOM events as well as more application-specific events via the `EventAggregator`.

<h3 id="dom-events"><a href="#dom-events">DOM Events</a></h3>

DOM events should be used when UI-specific messages need to be sent. They should not be used for application-specific messages. Aurelia doesn't add any functionality beyond the DOM for UI events (yet). Any Custom Attribute or Element can have its associated `Element` injected into its constructor. You can then use the `Element` to trigger events. To learn more about creating and triggering custom DOM events, [please read this article](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events).

<h3 id="the-event-aggregator"><a href="#the-event-aggregator">The Event Aggregator</a></h3>

If you need loosely coupled application-events, you want to use the `EventAggregator`. Its streamlined pub/sub interface makes it ideal for a wide range of messaging scenarios.

The Event Aggregator can publish events to a message channel or it can publish strongly-typed messages. Let's look at publishing to channels first:

```javascript
import {EventAggregator} from 'aurelia-event-aggregator';

export class APublisher{
    static inject = [EventAggregator];
    constructor(eventAggregator){
        this.eventAggregator = eventAggregator;
    }

    publish(){
        var payload = {}; //any object
        this.eventAggregator.publish('channel name here', payload);
    }
}
```

We begin by having the DI provide us with the singleton Event Aggregator. Next we call its `publish` method, passing it the message channel name and the data payload to send on that channel. Here's how a subscriber would set themself up to receive this:

```javascript
import {EventAggregator} from 'aurelia-event-aggregator';

export class ASubscriber{
    static inject = [EventAggregator];
    constructor(eventAggregator){
        this.eventAggregator = eventAggregator;
    }

    subscribe(){
        this.eventAggregator.subscribe('channel name here', payload => {
            //do something with the payload here
        });
    }
}
```

As you can see, they use the same channel name, but provide a callback, which will be invoked for every message sent on the channel.

Alternatively, you can publish and subscribe to strongly-typed messages. Here's an example publisher:

```javascript
export class SomeMessage{ }
```

```javascript
import {EventAggregator} from 'aurelia-event-aggregator';
import {SomeMessage} from './some-message';

export class APublisher{
    static inject = [EventAggregator];
    constructor(eventAggregator){
        this.eventAggregator = eventAggregator;
    }

    publish(){
        this.eventAggregator.publish(new SomeMessage());
    }
}
```

In this case, we publish an instance of a particular message type. Here's a sample subscriber:

```javascript
import {EventAggregator} from 'aurelia-event-aggregator';
import {SomeMessage} from './some-message';

export class ASubscriber{
    static inject = [EventAggregator];
    constructor(eventAggregator){
        this.eventAggregator = eventAggregator;
    }

    subscribe(){
        this.eventAggregator.subscribe(SomeMessage, message => {
            //do something with the message here
        });
    }
}
```

The subscriber will be called any time an instance of `SomeMessage` is published. Subscription is polymorphic, so if a subclass of SomeMessage is published, this subscriber will be notified as well.

>**Note:** All forms of the `subscribe` method return a _dispose function_. You can call this function to dispose of the subscription and discontinue receiving messages. A good place to dispose is either in a view-model's `deactivate` callback if it is managed by a router, or in its `detached` callback if it is any other view-model.

<h2 id="http-client"><a href="#http-client">HTTP Client</a></h2>

As a convenience, Aurelia includes a basic `HttpClient` to provide a comfortable interface to the browser's `XMLHttpRequest` object. `HttpClient` is not included in the modules that Aurelia's bootstrapper installs, since it's completely optional and many apps may choose to use a different strategy for data retrieval. So, if you want to use it, first you must install it with the following command:

```shell
jspm install aurelia-http-client
```

Then you can use it like this:

```javascript
import {HttpClient} from 'aurelia-http-client';

export class WebAPI {
    static inject = [HttpClient];
    constructor(http){
        this.http = http;
    }

    getAllContacts(){
        return this.http.get('url goes here');
    }
}

```

The `HttpClient` has the following implementation:

```javascript
export class HttpClient {
  configure(fn){
    var builder = new RequestBuilder(this);
    fn(builder);
    this.requestTransformers = builder.transformers;
    return this;
  }

  createRequest(url){
    let builder = new RequestBuilder(this);

    if(url) {
      builder.withUrl(url);
    }

    return builder;
  }

  delete(url){
    return this.createRequest(url).asDelete().send();
  }

  get(url){
    return this.createRequest(url).asGet().send();
  }

  head(url){
    return this.createRequest(url).asHead().send();
  }

  jsonp(url, callbackParameterName='jsoncallback'){
    return this.createRequest(url).asJsonp(callbackParameterName).send();
  }

  options(url){
    return this.createRequest(url).asOptions().send();
  }

  put(url, content){
    return this.createRequest(url).asPut().withContent(content).send();
  }

  patch(url, content){
    return this.createRequest(url).asPatch().withContent(content).send();
  }

  post(url, content){
    return this.createRequest(url).asPost().withContent(content).send();
  }
}
```

As you can see, it provides convenience methods for all the standard verbs as well as `jsonp`. Each of these methods sends an `HttpRequestMessage` except `jsonp` which sends a `JSONPRequestMessage`. The result of sending a message is a `Promise` for an `HttpResponseMessage`.

The `HttpResponseMessage` has the following properties:

* `response` - Returns the raw content sent from the server.
* `responseType` - The expected response type.
* `content` - Formats the raw `response` content based on the `responseType` and returns it.
* `headers` - Returns a `Headers` object with the parsed header data.
* `statusCode` - The server's response status code.
* `statusText` - The server's textual status message.
* `isSuccess` - Indicates whether or not the status code falls within the success range.
* `reviver` - A function used to transform the raw `response` content.
* `requestMessage` - A reference to the original request message.

> **Note:** By default, the `HttpClient` assumes you are expecting a JSON responseType.

<h3 id="interceptors"><a href="#interceptors">Interceptors</a></h3>

It is possible to hook into requests and responses with interceptors.

```javascript
class RequestInterceptor {
  request(message) {
    // do something with the message
    return message;
  }

  requestError(error) {
    throw error; // or return a (Http/Jsonp)RequestMessage to recover from the error
  }
}

class ResponseInterceptor {
  response(message) {
    // do something with the message
    return message;
  }

  responseError(error) {
    throw error; // or return an HttpResponseMessage to recover from the error
  }
}

var client = new HttpClient();
  .configure(x => {
    x.withInterceptor(new RequestInterceptor());
    x.withInterceptor(new ResponseInterceptor());
  });i
```

> **Note:** It is important to realize that all interceptors used with a client form a chain. The return value of an intercept method is passed on as the argument to the next. Interceptors are called in the order they were added.


There are two other apis that are worth noting. You can use `configure` to access a fluent api for configuring all requests sent by the client. You can also use `createRequest` to custom configure individual requests. Here's an example of configuration:

```javascript
var client = new HttpClient()
  .configure(x => {
    x.withBaseUrl('http://aurelia.io');
    x.withHeader('Authorization', 'bearer 123');
  });

client.get('some/cool/path');
```

In this case, all requests from the client will have the baseUrl of 'http://aurelia.io' and will have the specified Authorization header. The same API is available via the request builder. So, you can accomplish the same thing on an individual request like this:

```javascript
var client = new HttpClient();

client.createRequest('some/cool/path')
  .asGet()
  .withBaseUrl('http://aurelia.io')
  .withHeader('Authorization', 'bearer 123')
  .send();
```

The fluent API has the following chainable methods: `asDelete()`, `asGet()`, `asHead()`, `asOptions()`, `asPatch()`, `asPost()`, `asPut()`, `asJsonp()`, `withUrl()`, `withBaseUrl()`, `withContent()`, `withParams()`, `withResponseType()`, `withTimeout()`, `withHeader()`, `withCredentials()`, `withReviver()`, `withReplacer()`, `withProgressCallback()`, and `withCallbackParameterName()`.

>**Note:** We are working on a more moden HttpClient located in the `aurelia-fetch-client` repo. That is the client that the getting started guide now uses. We recommend using it, since it is based on the upcoming Fetch standard. It is still a work in progress. Docs on it will be coming in the near future.

<h2 id="debugging"><a href="#debugging">Debugging</a></h2>

During debug mode Aurelia will output various information to the browser's console window. However, this is not always enough information if you are having an issue with a binding expression or a view. To help in these situations Aurelia provides two Custom Attributes: `compile-spy` and `view-spy`. `compile-spy` can be placed on any element to have it emit the View Compiler's `TargetInstruction` into the debug console, giving you insight into all the parsed bindings, behaviors and event handers for the targeted element. The `view-spy` can be placed on any HTML element in a view to emit the View instance to the debug console, giving you insight into the live View instance, including all child views, live bindings, behaviors and more.

<h2 id="customization"><a href="#customization">Customization</a></h2>

<h3 id="view-and-view-model-conventions"><a href="#view-and-view-model-conventions">View and View-Model Conventions</a></h3>

How are views and view-models linked? Our simple convention is based on module id. If you've got a view-model with id (essentially path) './foo/bar/baz' then that will map to `./foo/bar/baz.js` and `./foo/bar/baz.html` by default. Suppose you want to follow a different convention though. What if all your view-models live in a `view-models` folder and you want their views to live in a `views` folder? How would you do that? In order to do this, you want to change the behavior of the Conventional View Strategy. Here's how you do it:

```javascript
import {ConventionalViewStrategy} from 'aurelia-framework';

ConventionalViewStrategy.convertModuleIdToViewUrl = function(moduleId){
  var id = (moduleId.endsWith('.js') || moduleId.endsWith('.ts')) ? moduleId.substring(0, moduleId.length - 3) : moduleId;
  return id + '.html';
}
```

You should execute this code as part of your bootstrapping configuration logic so that it takes effect before any Custom Elements are loaded. This will affect *everything* including custom elements. So, if you need or want those to act differently, you will need to account for that in your implementation of `convertModuleIdToViewUrl`.

> **Note:** This is an example of why 3rd party plugin authors should not rely on conventions. Developers may change these conventions in order to fit the needs of their own app.
