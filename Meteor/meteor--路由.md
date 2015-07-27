##### 路由

* 添加[Iron Router](https://github.com/EventedMind/iron-router)包

`meteor add iron:router`

###### 路由相关词汇

* 路由规则（Route）：路由规则是路由的基本元素。它的工作就是当用户访问 App 的某个 URL 的时候，告诉 App 应该做什么，返回什么东西。
* 路径（Path）：路径是访问 App 的 URL。它可以是静态的（/terms_of_service）或者动态的（/posts/xyz），甚至还可以包含查询参数（/search?keyword=meteor）。
目录（Segment）：路径的一部分，使用正斜杠（/）进行分隔。
* Hooks：Hooks 是可以执行在路由之前，之后，甚至是路由正在进行的时候。一个典型的例子是，在显示一个页面之前检测用户是否拥有这个权限。
* 过滤器（Filter）：过滤器类似于 Hooks ，为一个或者多个路由规则定义的全局过滤器。
* 路由模板（Route Template）：每个路由规则指向的 Meteor 模板。如果你不指定，路由器将会默认去寻找一个具有相同名称的模板。
* 布局（Layout）：你可以想象成一个数码相框的布局。它们包含所有的 HTML 代码放置在当前的模板中，即使模板发生改变它们也不会变。
* 控制器（Controller）：有时候，你会发现很多你的模板都在重复使用一些参数。为了不重复你的代码，你可以让这些路由规则继承一个路由控制器（Routing Controller）去包含所有的路由逻辑。

-----

* 创建layout文件： `client/templates/application/layout.html`

```
<template name="layout">
  <div class="container">
    <header class="navbar navbar-default" role="navigation">
      <div class="navbar-header">
        <a class="navbar-brand" href="/">Microscope</a>
      </div>
    </header>
    <div id="main" class="row-fluid">
      <!-- route templates -->
      {{> yield}}
    </div>
  </div>
</template>
```

* 创建路由文件: `lib/router.js` 

```
Router.configure({
  layoutTemplate: 'layout'
});

Router.route('/', {name: 'postsList'});
```

默认情况下，Iron Router 会为这个路由规则，指定相同名字的模板。而如果路径（path 参数）没有指定，它也会根据路由规则的名字，去指定同样名字的路径。举个例子，在上面的设置中，如果我们不提供 path 参数，那么访问 /postsList 将会自动获取到 postList 模板。

使用`{{pathFor}}`的Spacebars helper，也可以指定路由路径。比如：
```
<a class="navbar-brand" href="{{pathFor 'postsList'}}">Microscope</a>
```

* 等待数据

路由配置waitOn，可以等待数据返回再跳转。

```
Router.configure({
  layoutTemplate: 'layout',
  waitOn: function() { return Meteor.subscribe('posts'); }
});

Router.route('/', {name: 'postsList'});
```

还可以添加过渡的loading模板

```
Router.configure({
  layoutTemplate: 'layout',
  loadingTemplate: 'loading',
  waitOn: function() { return Meteor.subscribe('posts'); }
});

Router.route('/', {name: 'postsList'});
```

一般会添加spin包:

`meteor add sacha:spin`

然后再`client/templates/includes`添加loading模板：

```
<template name="loading">
  {{>spinner}}
</template>
```

`{{> spinner}}`是spin包中得一个模板标签。

-----

* 路由到一个特定的帖子

新建模板`client/templates/posts/post_page.html`:

```
<template name="postPage">
  {{> postItem}}
</template>
```

然后在`lib/router.js`添加路由规则：

```
Router.route('/posts/:_id', {
  name: 'postPage',
  data: function() { return Posts.findOne(this.params._id); }
});
```

接着提供路由生成的链接，在`client/templates/posts/post_item.html`: 

```
<template name="postItem">
  <div class="post">
    <div class="post-content">
      <h3><a href="{{url}}">{{title}}</a><span>{{domain}}</span></h3>
    </div>
    <a href="{{pathFor 'postPage'}}" class="discuss btn btn-default">Discuss</a>
  </div>
</template>
```

`<a href="{{pathFor 'postPage'}}">`等同于`<a href="/posts/{{_id}}">`

这里有些Iron Router的黑魔法，我们并没有传递`_id`，但是，首先通过路由配置知道需要`_id`，然后会在`{{pathFor 'postPage'}}`的上下文环境中（即`this`对象）寻找`_id`。

我们也可以通过传递 Helper 的第二个参数，来指定`_id`所在的`this`对象: `{{pathFor 'postPage' someOtherPost}}`。

-----

* 添加404模板

新建`client/templates/application/not_found.html`：

```
<template name="notFound">
  <div class="not-found jumbotron">
    <h2>404</h2>
    <p>Sorry, we couldn't find a page at this address. 抱歉，我们无法找到该页面。</p>
  </div>
</template>
```

添加路由规则:

```
Router.configure({
  layoutTemplate: 'layout',
  loadingTemplate: 'loading',
  notFoundTemplate: 'notFound',
  waitOn: function() { return Meteor.subscribe('posts'); }  
});
Router.onBeforeAction('dataNotFound', {only: 'postPage'});
```

-----

* 数据源

通过设置模板的数据源，你可以在模板 helper 里面控制 this 的值。

这个工作通常会隐式地被 {{#each}} 迭代器完成，它会自动设置对应的数据源到每个正在迭代的当前项中：

```
{{#each widgets}}
  {{> widgetItem}}
{{/each}}
```

当然我们也可以使用 {{#with}} 去显式地操作，它就像简单地说“拿这个对象，提供给下面的模板应用”。例如，我们可以这样写：

```
{{#with myWidget}}
  {{> widgetPage}}
{{/with}}
```

因此通过传递数据源作为参数给模板调用也可以实现相同的效果，所以前面的代码块可以重写为：

```
{{> widgetPage myWidget}}
```

##### 会话(Session)

Session 是一个全局的响应式数据存储。它全局性的意思是全局的单例对象：这个 Session 对象在全局都是可被访问到。全局变量通常被认为不是一件什么好事，不过Session 可以作为中央通信总线用于项目的不同地方。

通过 Session.get('mySessionProperty'); 你可以重新读取数据的内容，这是一个响应式的数据源，这意味着如果你把它放在一个 Helper 里面，你会看到 Helper 根据 Session 变量的改变而响应式地改变它的输出。

Session 对象不在用户之间共享，甚至在浏览器标签之间。

* 自动运行（Autorun）

每一次 [autorun](http://docs.meteor.com/#Tracker_autorun) 上下文中的响应式数据源发生变化的时候，autorun 函数就会自动运行。

`Tracker.autorun( function() { console.log('Value is: ' + Session.get('pageTitle')`

上面的代码会先输出一次`Value is: A brand new title`，然后每当我们`Session.set('pageTitle', 'Yet another value');`的时候，随着 Session 变量的改变， autorun 知道它必须重新运行自己的代码，并把新的值重新输出到控制台，输出`Value is: Yet another value`.

* 动态代码重载技术（Hot Code Reload）

Meteor 的动态代码重载技术（HCR）：当我们修改并保存一个源代码的文件后，Meteor 会立刻检测到变化，直接重启正在运行的 Meteor 服务器，并通知每个客户端重新加载该页面。

动态代码重载与刷新重载页面有一个区别是：

如果我们手动去重载浏览器窗口，自然就会丢失我们的 Session 变量（因为这将会创建一个新的会话）。另一方面，如果我们是引发动态代码重载（即，通过修改并保存我们的源文件）去重新加载页面，Session 变量却仍然存在。
