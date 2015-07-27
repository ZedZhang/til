##### 响应式

meteor讲的响应式，并不是布局上的响应式，而是数据上的响应式。

集合从根本上改变你的应用程序的数据处理方式。从而不必手动检查数据更改（例如，通过一个 AJAX 调用），再根据这些变化去修改 HTML 页面，Meteor 可以随时检测到数据的更改，并将它无缝地应用到你的用户界面上。

这个实时性的方法是通过使用 .observe() ，当指向数据的指针发生改变时就会触发回调。我们可以通过这些回调去更改 DOM （我们的网页呈现的 HTML ）。就像这样的代码：

```
Posts.find().observe({
  added: function(post) {
    // when 'added' callback fires, add HTML element
    $('ul').append('<li id="' + post._id + '">' + post.title + '</li>');
  },
  changed: function(post) {
    // when 'changed' callback fires, modify HTML element's text
    $('ul li#' + post._id).text(post.title);
  },
  removed: function(post) {
    // when 'removed' callback fires, remove HTML element
    $('ul li#' + post._id).remove();
  }
});
```

* 声明式方法

Meteor 为我们提供了一个更好的办法：声明式方法，它的核心就是响应式。这种声明让我们定义了对象之间的关系，并让他们保持同步，而我们就不必为每个的可能发生的修改去指定相应的行为。

通过声明式方法去声明我们该如何基于响应式数据去呈现 HTML ，这样 Meteor 就可以完成对这些数据的监控工作，并且把用户界面直接与数据进行绑定。

总的来说，去代替 observe 的回调，Meteor 可以让我们这些写：

```
<template name="postsList">
  <ul>
    {{#each posts}}
      <li>{{title}}</li>
    {{/each}}
  </ul>
</template>
```

然后获取我们的帖子列表：

```
Template.postsList.helpers({
  posts: function() {
    return Posts.find();
  }
});
```

在后台，其实 Meteor 替我们使用了 observe() 的回调方法，并当响应式数据被更改的时候，对相关页面进行重新的渲染。

* Meteor 的依赖跟踪：Computations

并不是所有的代码在 Meteor App 里面都是响应式的，响应式只是在特定区域的代码中发生，我们称这些区域为 **Computations** 。

换句话说， Computations 代码块是根据响应式数据的变化去运行。如果你有一个响应式数据源（例如，一个 Session 变量）并且希望及时去响应它，你需要建立一个 Computations。

请注意，你一般不需要显式地做到这一点，因为 Meteor 已经让每个模板去呈现它自己的 Computation （意思就是模板 Helper 中的代码和回调函数默认都是响应式的）。

Computations 用到的响应式数据源都会被它跟踪，这样就可以知道响应式数据什么时候发生变化。这是通过 Computations 中的 invalidate() 方法实现的。

Computations 一般只是简单地用来判断页面上的无效内容，这通常是发生在模板的 Computations（尽管模板 Computations 可能也会去做一些让页面更有效的工作）。当然如果你需要，你也可以使用更多 Computations 的功能，不过在实际中可能比较少用到。

* 建立一个 Computations

只需要在 Tracker.autorun 方法中加上需要的代码块，让它变成响应式的 Computations ：

```
Meteor.startup(function() {
  Tracker.autorun(function() {
    console.log('There are ' + Posts.find().count() + ' posts');
  });
});
```

在后台，autorun 会创建一个 Computation ，当数据源发生变化的时候就会自动重新运行。

```
> Posts.insert({title: 'New Post'});
There are 4 posts.
```

