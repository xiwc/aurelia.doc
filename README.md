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

_思考下这意味着什么?_


单向数据绑定意思是JavaScript视图模型中的改变反映到视图中, 但是视图的改变不会反映到视图模型中. 双向数据绑定意思是改变的映射是双向的. `.bind`会使明智的使用默认行为, 如果你绑定到一个表单`form`元素的值`value`属性, 并且你可能希望表单的改变反映到你的视图模型中. 对于其他的情形会使用单向数据绑定, 特别的原因是,在大多数情形, 双向数据绑定对于非表单元素是没有意义的. 这里是一个小的使用`.bind`的数据绑定的例子:

```markup
<input type="text" value.bind="firstName">
<a href.bind="url">Aurelia</a>
```

在上面的例子中, `input`会绑定它的`value`到视图模型的`firstName`属性上. 改变`firstName`属性将会更新`input.value`, 并且改变`input.value`也会更新`firstName`属性. 另一方面, `a`将会绑定它的`href`到视图模型的`url`属性. 一旦`url`属性改变, 就会反映到`a`的`href`上, 而另一个方向不会.

你可以一直明确的使用`.one-way`或者`.two-way`来代替`.bind`. 一个常见的使用情形是, 当被Web组件需要作为输入类型功能的组件. 所以你可以假想做下面的事情: 

```markup
<markdown-editor value.two-way="markdown"></markdown-editor>
```

为了优化性能和减少CPU和内存的使用率, 你可以有选择的使用`one-time`绑定命令, 去反映视图模型中的数据改变到视图中,仅仅是一次性的. 这将发生在初始化绑定阶段, 在此之后, 不会再发生同步.

<h4 id="event-modes"><a href="#event-modes">代理, 触发器, 呼叫(delegate, trigger & call)</a></h4>

绑定命令不仅仅连接视图模型属性到视图元素属性, 而且可以被用来触发行为. 例如, 如果你想调用一个视图模型中的方法在按钮被点击后, 你可以使用`trigger`命令像下面这样: 

```markup
<button click.trigger="sayHello()">Say Hello</button>
```

当按钮被点击, 视图模型中的`sayHello`方法会被调用. 也就是说, 像这样添加事件处理函数到每一个单独的元素上不是很高效的, 所以经常你会想使用事件代理. 使用`.delegate`命令来完成代理. 下面是一个简单的使用事件代理的例子:

```markup
<button click.delegate="sayHello()">Say Hello</button>
```

`$event`属性可以作为一个参数传递到一个delegate/trigger函数调用中, 如果你需要访问事件的对象.


```markup
<button click.delegate="sayHello($event)">Say Hello</button>
```

>  **注意:** 如果你不熟悉事件代理, 它是使用DOM事件的冒泡特性. 当使用`.delegate`时, 一个单独的事件处理函数被附加到文档对象上, 而不是附加到每一个元素上. 当元素事件被触发, 它会冒泡DOM树直到它到达文档对象, 在那里它会被处理. 这是更高内存利用率的事件处理方式, 并且这是被推荐使用.

>  **注意:** 在一个*closed* ShadowDOM中事件代理是不起作用的. 它可以毫无顾虑的在一个open ShadowDOM中起作用.

在DOM事件中这大都是正常工作的. 偶尔你或许有一个Aurelia自定义属性或者元素想直接引用你的一个函数, 以便于在晚点可以手动的调用. 要传递一个函数引用, 使用`.call`绑定(因为这个属性会将来被 _call_):

```markup
<button touch.call="sayHello()">Say Hello</button>
```

现在这个自定义的属性`touch`引用一个函数,让它可以调用你的`sayHello`函数代码. 依赖于者天然的实现, 你可以从调用者接受数据. 它像trigger/delegate一样工作, 可以提供一个 `$event`对象参数.

<h4 id="string-interpolation"><a href="#string-interpolation">字符串插入(string interpolation)</a></h4>

有时你需要直接绑定属性到文档的内容中, 或者间插它们到属性值中. 为了这样, 你可以使用字符串插入语法`${expression}`. 字符串插入是单向绑定, 输出被转换层字符串. 下面是一个例子: 

```markup
<span>${fullName}</span>
```

`fullName`属性会被直接内嵌到span的内容中. 你也可以使用它处理css class的绑定, 像这样:

```markup
<div class="dot ${color} ${isHappy ? 'green' : 'red'}"></div>
```

在这个代码片段中"dot"是一个静态呈现的class, "green" class被呈现仅仅当`isHappy`为true, 否则"red" class会被呈现. 另外的, 不管`color`是什么值, 它都会被添加作为一个class.

> **注意:** 你可以使用简单的表达式在你的绑定中. 不要尝试太过花哨的做法, 你不想在视图中编码. 你仅仅想建立视图到视模型的连接.

<h4 id="ref"><a href="#ref">引用(ref)</a></h4>

除了命令和插入, 绑定语言赞成使用特性的属性: `ref`. 通过使用`ref`你可以为一个元素创建一个本地的名称, 以便于它可以在其他的绑定表达式中被应用. 它也可以做设置为视图模型的一个属性., 以便于你可以通过代码访问它. 下面是一个巧妙使用`ref`的例子:

```markup
<input type="text" ref="name"> ${name.value}
```

你也可以使用`ref`绑定命令获取Aurelia自定义元素或者属性背后的视图模型实例. 通过使用这种技术, 你可以相互连接不同组件:

```markup
<producer producer.ref="producerVM"></producer>
<consumer input.bind="producerVM.output"></consumer>
```

`producer.ref="producerVM"`创建`producer`自定义元素的视图模型的一个别名, 以便于可以在其他地方使用, 传递给另外的自定义元素, 或者使用视图模型的属性. 因此在上面例子的第二行, `consumer`有一个叫`input`的属性绑定到`producer`视图模型的`output`属性上. 有一些不同的方式在引用元素和视图模型中使用`ref`:

* `attribute-name.ref="someIdentifier"`- 创建一个自定义属性类实例的引用
* `element-name.ref="someIdentifier"`- 创建一个自定义元素类实例的引用
* `ref="someIdentifier"` - 创建一个DOM树中的HTMLElement的引用

<h4 id="select-elements"><a href="#select-elements">选择元素(select elements)</a></h4>

`value.bind`在HTMLSelectElement上有者特别的行为, 为支持元素的单选和多选模式绑定到对象.

一个典型的被渲染的选择元素使用`value.bind` 和 `repeat`组合是像这样:

```markup
<select value.bind="favoriteColor">
    <option>Select A Color</option>
    <option repeat.for="color of colors" value.bind="color">${color}</option>
</select>
```

有时,你想绑定对象实例而不是字符串. 下面是一个构建选择元素的标记, 它使用假设的employee对象数组:

```markup
<select value.bind="employeeOfTheMonth">
  <option>Select An Employee</option>
  <option repeat.for="employee of employees" model.bind="employee">${employee.fullName}</option>
</select>
```

这个例子和上个例子主要的不同点是, 我们保存选项的值在一个特性的属性`model`中, 代替仅仅能够接受字符串的选项元素的`value`属性.

<h4 id="multi-select-elements"><a href="#multi-select-elements">多选元素(multi select elements)</a></h4>

你可以绑定多选元素的值到一个数组属性中在一个多选场景中. 下面是你怎么绑定到一个字符串数组`favoriteColors`:

```markup
<select value.bind="favoriteColors" multiple>
    <option repeat.for="color of colors" value.bind="color">${color}</option>
</select>
```

这同样可以工作于对象数组:

```markup
<select value.bind="favoriteEmployees" multiple>
  <option repeat.for="employee of employees" model.bind="employee">${employee.fullName}</option>
</select>
```

<h4 id="radios"><a href="#radios">(单选)radios</a></h4>

HTMLInputElement上的`checked.bind`有一个特别的行为, 它支持非boolean类型的值, 例如字符串和对象.

一个典型的单选按钮组使用`value.bind` 和 `repeat`组合来渲染, 像下面这样:

```markup
<label repeat.for="color of colors">
  <input type="radio" name="clrs" value.bind="color" checked.bind="$parent.favoriteColor" />
  ${color}
</label>
```

有时你想使用对象实例而不是字符串, 下面是一个使用假想的employee对象数组构建单选按钮组的标记:

```markup
<label repeat.for="employee of employees">
  <input type="radio" name="emps" model.bind="employee" checked.bind="$parent.employeeOfTheMonth" />
  ${employee.fullName}
</label>
```

这个例子和先前的例子主要不同点是, 我们储存input的值在一个特别的属性(`model`)上, 替代仅仅接受字符串的input元素的`value`属性.

你也可以绑定一个单选组到一个boolean属性, 像这样:

```markup
<label><input type="radio" name="tacos" model.bind="null" checked.bind="likesTacos" />Unanswered</label>
<label><input type="radio" name="tacos" model.bind="true" checked.bind="likesTacos" />Yes</label>
<label><input type="radio" name="tacos" model.bind="false" checked.bind="likesTacos" />No</label>
```

<h4 id="checkboxes"><a href="#checkboxes">复选(checkboxes)</a></h4>

为了更好的支持多选场景, Aurelia允许绑定一个input元素的checked属性到一个数组. 下面展示绑定到一个字符串数组`favoriteColors`:

```markup
<label repeat.for="color of colors">
  <input type="checkbox" value.bind="color" checked.bind="$parent.favoriteColors" />
  ${color}
</label>
```

对于对象数组也就是可以工作的:

```markup
<label repeat.for="employee of employees">
  <input type="checkbox" model.bind="employee" checked.bind="$parent.favoriteEmployees" />
  ${employee.fullName}
</label>
```

你也可以绑定每一个checkboxes到它的boolean属性, 像这样:

```markup
<li><label><input type="checkbox" checked.bind="wantsFudge" />Fudge</label></li>
<li><label><input type="checkbox" checked.bind="wantsSprinkles" />Sprinkles</label></li>
<li><label><input type="checkbox" checked.bind="wantsCherry" />Cherry</label></li>
```

<h4 id="innerhtml"><a href="#innerhtml">内嵌HTML(innerHTML)</a></h4>

你能够绑定一个元素的`innerHTML`属性:

``` markup
<div innerhtml.bind="htmlProperty"></div>
<div innerhtml="${htmlProperty}"></div>
```

Aurelia提供一个简单的html清洁处理转换, 像下面这样:


``` markup
<div innerhtml.bind="htmlProperty | sanitizeHtml"></div>
<div innerhtml="${htmlProperty | sanitizeHtml}"></div>
```

我们鼓励使用更加完整的html清洁处理, 例如[sanitize-html](https://www.npmjs.com/package/sanitize-html). 下面教你怎么使用它构建转换器:


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

<h4 id="textcontent"><a href="#textcontent">文本内容(textContent)</a></h4>

你可以绑定元素的`textContent`属性:


``` markup
<div textcontent.bind="stringProperty"></div>
<div textcontent="${stringProperty}"></div>
```

在`contenteditable`元素上支持双向数据绑定:

``` markup
<div textcontent.bind="stringProperty" contenteditable="true"></div>
```

<h4 id="style"><a href="#style">样式(style)</a></h4>

你可以绑定一个css字符串或者对象到一个元素的`style`属性:

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

使用`style`属性的别名`css`, 使用字符串篡改保证你的引用兼容Internet Explorer:

``` markup
<!-- good: -->
<div css="width: ${width}px; height: ${height}px;"></div>

<!-- incompatible with Internet Explorer: -->
<div style="width: ${width}px; height: ${height}px;"></div>
```

<h4 id="adaptive-binding"><a href="#adaptive-binding">适配的绑定(Adaptive Binding)</a></h4>

Aurelia会从若干策略中选择一个适配的绑定系统当决定怎样去最有效的观察监测变化. 想了解更多的关于他的工作机制, 检出[this post](http://blog.durandal.io/2015/04/03/aurelia-adaptive-binding/). 对于大多数情况你不需要考虑这方面的细节, 然而它可以帮助你发现导致低效使用的绑定系统.

**第一条要意识到的是,计算的属性(属性的getter方法)是使用脏检查机制观察监测.** 更加有效率的策略, 例如Object.observe和属性重写是不兼容对于这些类型的属性.


在今天的浏览器环境dirty-checking是个被需要的坏孩子. 很少的浏览器支持Object.observe在当前文档写作的时候. Aurelia的dirty-checking机制是类似于使用[Polymer](https://www.polymer-project.org/). 它是非常有效的, 并且使用Aurelia的微任务队列(micro-task-queue)去批量更新DOM.

很少的使用dirty-checking的绑定系统不会导致系统的性能问题. 大量的使用时有可能的. 幸运的是你可以避免dirty-checking通过简单的计算属性. 考虑下面的'fullName'属性例子: 

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

我们使用`@computedFrom`装饰器提供对于Aurelia绑定系统的暗示. 绑定系统这样就会知道仅仅去监测`fullName`的变化, 当 `firstName` 或者 `lastName` 发生变化时.

流行dirty-checking的工作机制也是很重要的. 当一个属性使用"dirty-checked", 绑定系统会定期的检查属性当前的值和先前监测的值是否匹配. 默认的检查周期是120毫秒. 这意味着你的属性的getter函数可能被非常频繁的调用, 所以你的方法应该尽可能的高效. 你应该避免无用的返回对象实例或者数组. 思考下下面的视图: 

```markup
<template>
  <label for="search">Search Issues:</label>
  <input id="search" type="text" value.bind="searchText" />
  <ul>
    <li repeat.for="issue of filteredIssues">${issue.abstract}</li>
  </ul>
</template>
```

天真的视图模型的实现:

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

改进的视图模型实现:

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

<h3 id="html-extensions"><a href="#html-extensions">HTML扩展(HTML Extensions)</a></h3>

除了数据绑定, 你也可以使用Aurelia的HTML扩展. 有两种类型:

* 自定义元素 - 扩展HTML的标签! 你的自定义元素可以有它自己的视图(可以使用数据绑定和其他的html扩展), 并且有选择的使用[ShadowDOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)(甚至于浏览器不支持它).
* 自定义属性 - 扩展HTML新的属性, 它可以被添加到已经存在或者自定义的元素上. 这些属性添加新的行为到元素上.

自然的, 它们都无缝的和数据绑定一起发挥作用. 让我们看看Aurelia为你提供的一些自定义元素和属性, 它们是全局的对于每一个视图.

<h4 id="show"><a href="#show">显示(show)</a></h4>

`show`自定义属性允许你有条件的展示一个HTML元素. 如果属性值为`true`元素会显示, 反之会隐藏. 这个属性不会从DOM中添加或者删除元素, 而仅仅是改变它的可见性. 下面是个例子:

```markup
<div show.bind="isSaving" class="spinner"></div>
```

当`isSaving`属性为true, `div`会可见, 否则会隐藏.

<h4 id="if"><a href="#if">如果(if)</a></h4>

`if`自定义属性允许你有条件的添加或者删除一个HTML元素. 如果属性值为true, 元素会呈现在DOM中, 反之, 不会被呈现.

```markup
<div if.bind="isSaving" class="spinner"></div>
```

这个例子看起来和上面的`show`例子相似. 不同点在于, 如果绑定表达式计算值为false, `div`会被从DOM中删除, 而不是隐藏.

如果你需要有条件的添加或者移除一组元素, 你不能放置`if`属性在一个父元素上, 你可以包裹这些元素在一个template标签内. 下面是描述的例子:

```markup
<template if.bind="hasErrors">
    <i class="icon error"></i>
    ${errorMessage}
</template>
```

> **注意:** 这是很重要的, 你不应该添加`if`行为围绕着`<content>`元素. ShadowDOM不支持按照你期望的那样动态的添加这些元素. 作为替代, 在一个父元素上使用`show`行为.

<h4 id="repeat"><a href="#repeat">重复(repeat)</a></h4>

`repeat`自定义属性允许你多次渲染一个模板, 数组的每一个项目渲染一次. 下面是个渲染自定义名称列表的例子:

```markup
<ul>
    <li repeat.for="customer of customers">${customer.fullName}</li>
</ul>
```

一个重要注意点是, repeat属性和`.for`绑定命名一起结合工作. 这个绑定命名解释一个特别的语法在"item of collection"的形式中, "item"是一个本地的名称, 会被在template中使用, "collection"是一个正常的计算为数组或者Map的绑定表达式.

说到Maps, 下面是一个怎么绑定ES6 Map的例子:

```markup
<ul>
  <li repeat.for="[id, customer] of customers">${id} ${customer.fullName}</li>
</ul>
```

如果你想替代遍历一个集合去遍历一个特别的数字次数, 你可以使用"i of count"这种替代语法, "i"是遍历的索引, "count"是一个可以计算为正整数的绑定表达式.

```markup
<ul>
  <li repeat.for="i of rating">*</li>
</ul>
```

> **注意:** 和`if`属性一样, 你也可以使用一个`template`标签, 去包裹一组没有父元素的集合元素.

每一个被`repeat`属性重复的项目有一些特别的上下文值用于绑定.

* `$parent` - 当前, 主视图模型的属性和方法对于重复的项目是不可见的. 我们在将来的更新进行补救. 同时, 你可以访问主视图模型通过`$parent`.
* `$index` - 数组中项目的索引.
* `$first` - True 如果项目是数组的第一个.
* `$last` - True 如果项目是数组的最后一个.
* `$even` - True 如果项目的索引数偶数.
* `$odd` - True 如果项目的索引数奇数.

<h4 id="compose"><a href="#compose">组合(compose)</a></h4>

`compose`自定义元素允许你动态的渲染你的UI到DOM中. 假设你有一个多样的数组项目, 但是每一个都有一个类型属性告诉你它代表什么. 你接着可以这样做:

```markup
<template repeat.for="item of items">
    <compose
      model.bind="item"
      view-model="widgets/${item.type}">
    </compose>
</template>
```

现在, 根据项目的 _type_ , `compose`元素会加载不同的视图模型(和视图), 并且渲染到DOM中. 如果视图模型有一个`activate`方法, `compose`元素会调用它, 并且传递给它`model`参数. `activate`方法甚至可以返回一个`Promise`, 这会延迟组合的处理直到一些异步任务完成后, 在正真的数据绑定和渲染到DOM之前.

`compose`元素也有一个`view`属性, 可以像`view-model`一样使用, 如果你不希望沿用标准的view/view-model约定. 如果你指定`view`, 但是没有指定`view-model`, 视图将会绑定到外围的上下文中.

```markup
<template repeat.for="item of items">
    <compose view="my-view.html"></compose>
</template>
```

如果你想绑定一个特别的对象, 当你仅仅使用`view`, 而不是齐全的视图模型(可能作为重复(repeat)的一部分), 你可以直接绑定 `view-model`. 然后你将能够直接的使用对象的属性在你的视图中.

```markup
<template>
    <div repeat.for="item of items">
      <compose view="my-view.html" view-model.bind="item">
    </div>
</template>
```

万一你想基于数据动态决定视图怎么办呢? 或者运行时的条件? 你可以通过在视图模型中实现`getViewStrategy()` 方法. 它可以返回一个视图的相对路径, 或者一个`ViewStrategy`实例来决定自定义视图的加载方式. 一个亮点是, 这个方法在`activate`回调之后执行, 所以你可以访问模型数据在决定加载什么视图的时候.


<h4 id="global-behavior"><a href="#global-behavior">全局行为(global-behavior)</a></h4>

它不是你将直接使用的HTML增强. 而是, 它需要结合自定义的绑定命令去动态的开启jQuery插件和HTML中相似的APIs声明. 让我们看一个例子帮助我们理解:

```markup
<div jquery.modal="show: true; keyboard.bind: allowKeyboard">...</div>
```

这个例子是基于[Bootstrap modal widget](http://getbootstrap.com/javascript/#modals). 在这个例子中, `modal` jQuery插件将会附着在`div`上, 并且会配置它的`show`选项为`true`, 设置`keyboard` 属性为视图模型的 `allowKeyboard`属性的值. 当包含它的视图解除绑定后, jQuery组件会被销毁.

组合特别的`global-behavior`和自定义语法的能力开启这些动态的能力. 这儿你看到的语法是基于本地的`style`属性的语法, 它列举属性和值用上面时尚的方式分隔. 注意, 你可以使用绑定命令(`.bind`)来传递视图模型中的数据到插件, 或者用`.call`来直接传递一个回调函数到插件中.

这里是它的工作机制:

当绑定系统看到它不认识的绑定命令, 它会动态的解释它, 属性名被映射到全局的绑定处理器, 它会解释绑定命令. 处理器可以使用它的值去创建一个选项对象, 然后传递给插件. 当视图被解除绑定, 处理器也可以清理插件. 在这种情形, jQuery处理器知道实例化插件的形式, 并且使用`destroy` 方法清理资源.

> **注意:** `global-behavior`有一个你必须配置的处理器. 它被默认配置成jQuery. 如果你不想要, 你可以把它们全部关闭, 但是开启它可以让你不用工作额外工作就可以容易的利用基本的jQuery插件.

<h2 id="routing"><a href="#routing">路由(Routing)</a></h2>

有很多不同的应用程序风格你可以被调用根据你的创造. 从导航应用, 到仪表面板, 到多文档界面, Aurelia都可以处理. 这些场景大都有一个主要的结构组件, 叫做客户端路由, 承担翻译url改变到应用程序的状态改变.

如果你已经阅读过开始指导, 你就会了解路由有两个部分, 第一个, 在你的视图模型中有一个`Router`. 它被配置路由导航的信息和导航控制. 第二个, 在视图中有一个`router-view`, 负责展示路由表示的当前界面状态.

让我们看一个配置例子:

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

我们开始于实现`configureRouter`方法. 我们可以可选的设置一个`title`属性, 去用于构建文档的标题, 但是最重要的部分是设置路由. 路由的`map`方法接受一个简单的JSON数据结构表示你的路由表. 最重要的两个属性是`route`(一个字符串或者字符串数组), 它定义路由的形式, `moduleId`, 指定为相对于你视图模型`relative`模块ID. 你也可以设置一个`name`属性, 用于在以后生成一个路由链接, 设置一个`title`属性, 用于生成文档标题, 设置一个`nav`属性, 标识是否该路由会被包含在导航模型中(它也可以是一个标识顺序的数字).  设置一个`href`属性, 用于绑定到 _navigation model_ .

所以, 对于路由形式你可以有那些配置选项?

* 静态路由
    - 例如`home`是严格的字符串匹配.
* 参数化路由
    - 例如`users/:id/detail`是字符串匹配并且解析一个`id`参数. 你的视图模型回调函数`activate`会传递一个包含`id`属性的对象作为参数,这个`id`就是从url中匹配得到的.
* 通配符路由
    - 例如`files*path`是匹配任何这种形式的字符串. 你的视图模型回调函数`activate`会传递一个包含`path`属性的对象作为参数,这个`path`就是通配符匹配得到的.

所有有`nav`属性的路由被装配到一个`navigation`数组. 这使得很容易通过数据绑定产生一个菜单结构. 另一个重要的绑定属性是`isNavigating`. 下面是一个简单的例子:


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

### 屏幕激活生命周期(The Screen Activation Lifecycle)

不管什么时候路由处理一个导航切换, 它会强制执行视图模型中的一个严格的生命周期, 它导航到哪里, 来自哪里. 有四个生命周期场景. 你可以选择性的进入通过实现适当的方法在你的视图模型中. 下面是生命周期回调方法的列表:

* `canActivate(params, routeConfig, navigationInstruction)` - Implement this hook if you want to control whether or not your view-model _can be navigated to_. Return a boolean value, a promise for a boolean value, or a navigation command.
* `activate(params, routeConfig, navigationInstruction)` - Implement this hook if you want to perform custom logic just before your view-model is displayed. You can optionally return a promise to tell the router to wait to bind and attach the view until after you finish your work.
* `canDeactivate()` - Implement this hook if you want to control whether or not the router _can navigate away_ from your view-model when moving to a new route. Return a boolean value, a promise for a boolean value, or a navigation command.
* `deactivate()` - Implement this hook if you want to perform custom logic when your view-model is being navigated away from. You can optionally return a promise to tell the router to wait until after your finish your work.

`params`对象会包含一个属性对应于每一个解析的路由参数, 同时也对应于每一个查询字符参数. `routeConfig`是你设置的原始的路由配置. `routeConfig` 会有一个新的 `navModel` 属性, 可用于改变文档标题, 当在你的视图模型中加载数据. 例如:

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

<h3 id="conventional-routing"><a href="#conventional-routing">常规路由(Conventional Routing)</a></h3>

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

<h3 id="customizing-the-navigation-pipeline"><a href="#customizing-the-navigation-pipeline">定制导航管道(Customizing the Navigation Pipeline)</a></h3>

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

<h3 id="configuring-push-state"><a href="#configuring-push-state">配置PushState(Configuring PushState)</a></h3>

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
* Use the Aurelia object during your bootstrapping phase to call `.globalResources(...resourcePaths)` to register extensions with global visibility in your application.
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
