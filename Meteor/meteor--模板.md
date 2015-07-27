##### 模板( Spacebars )

比如我们在html文件中插入了 `{{> postsList}}` 标签,它是 `postsList` 模板的插入点，现在我们来创建模板:

先在 `/client` 里面创建一个 `/templates` 目录

> 其实无论你把代码文件放在 `/client` 目录下的任何地方，Meteor 都可以找到它并且正确地进行编译。这意味着你永远都不需要手动编写 JavaScript 或 CSS 文件的调用路径。

在 `client/templates/posts` 目录中，创建 `posts_list.html` :

```
<template name="postsList">
  <div class="posts">
    {{#each posts}}
      {{> postItem}}
    {{/each}}
  </div>
</template>
```

和 `post_item.html` ：

```
<template name="postItem">
  <div class="post">
    <div class="post-content">
      <h3><a href="{{url}}">{{title}}</a><span>{{domain}}</span></h3>
    </div>
  </div>
</template>
```

Meteor 通过模板的`name="postsList"`属性，跟踪模板的位置，注意与实际的文件名不相关。

Spacebar 就是简单的 HTML 加上三件事情：**Inclusion**（有时也称作 “partial”）、**Expression** 和 **Block Helper**。

**Inclusion**： 通过 `{{> templateName}}` 标记，简单直接地告诉 Meteor 这部分需要用相同名称的模板来取代（在我们的例子中就是 postItem ）。

**Expression**：比如 `{{title}}` 标记，它要么是调用当前对象的属性，要么就是对应到当前模板管理器中定义的 helper 方法，并返回其方法值（后面会详细讨论）。

**Block Helper**：在模板中控制流程的特殊标签，如 `{{#each}}…{{/each}}` 或 `{{#if}}…{{/if}}` 。

##### 模板Helper( Controller中的Action? )

为简单起见，我们将采用与模板同名的方式来命名包含其 helper 的文件，区别是 `.js` 扩展名。那好让我们马上在 `/client/templates/posts` 目录下创建 `posts_list.js`:

```
var postsData = [
  {
    title: 'Introducing Telescope',
    url: 'http://sachagreif.com/introducing-telescope/'
  }, 
  {
    title: 'Meteor',
    url: 'http://meteor.com'
  }, 
  {
    title: 'The Meteor Book',
    url: 'http://themeteorbook.com'
  }
];
Template.postsList.helpers({
  posts: postsData
});
```

* `Template.postsList`指对应的模板名。
* `posts`模板中对应的expression
* `postsData`虚构的数据，用于填充模板`postItem`中对应的`title`和`url`。

假如我们加入一个`post_item.js`作为postItem的模板：

```
Template.postItem.helpers({
  domain: function() {
    var a = document.createElement('a');
    a.href = this.url;
    return a.hostname;
  }
});
```

* 在`posts_list.html` 模板中`{{#each}}` 代码块不仅遍历我们数组，它还在代码块范围内将 `this` 的值赋予被遍历的对象。
