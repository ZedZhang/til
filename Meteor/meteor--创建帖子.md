##### 创建帖子

* 构建新帖子的提交页面

首先为新帖子的提交页面定义一个路由，在`lib/router.js`添加：

```
Router.route('/submit', {name: 'postSubmit'});
```

然后再页面上添加该路由的链接，在header模板中添加：

```
<template name="header">
  <nav class="navbar navbar-default" role="navigation">
    <div class="container-fluid">
      <div class="navbar-header">
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navigation">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="{{pathFor 'postsList'}}">Microscope</a>
      </div>
      <div class="collapse navbar-collapse" id="navigation">
        <!--- 这一部分 ---->
        <ul class="nav navbar-nav">
          <li><a href="{{pathFor 'postSubmit'}}">Submit Post</a></li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
          {{> loginButtons}}
        </ul>
      </div>
    </div>
  </nav>
</template>
```

设置了这个路线就意味着如果用户浏览 /submit 的 URL 路径， Meteor 会显示 postSubmit 模板。 下面让我们来写这个模板`client/templates/posts/post_submit.html`吧：

```
<template name="postSubmit">
  <form class="main form">
    <div class="form-group">
      <label class="control-label" for="url">URL</label>
      <div class="controls">
          <input name="url" id="url" type="text" value="" placeholder="Your URL" class="form-control"/>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label" for="title">Title</label>
      <div class="controls">
          <input name="title" id="title" type="text" value="" placeholder="Name your post" class="form-control"/>
      </div>
    </div>
    <input type="submit" value="Submit" class="btn btn-primary"/>
  </form>
</template>
```

* 创建帖子

创建`client/templates/posts/post_submit.js`：

```
Template.postSubmit.events({
  'submit form': function(e) {
    e.preventDefault();

    var post = {
      url: $(e.target).find('[name=url]').val(),
      title: $(e.target).find('[name=title]').val()
    };

    post._id = Posts.insert(post);
    Router.go('postPage', post);
  }
});
```

* 添加一些安全检查

比如我们要求只有登录的用户才能创建帖子，meteor本身已经集成关于数据安全的模块，只是默认关闭而已，现在我们把它开启：

```
meteor remove insecure
```

没有了 insecure 包，从客户端插入帖子集合已经不再被允许了。我们需要给出一些明确的规则告诉 Meteor ，什么时候才能允许客户插入帖子，否则我们只能从服务端插入。

* 允许帖子插入

只需在`lib/collections/posts.js`中定义：

```
Posts = new Mongo.Collection('posts');

Posts.allow({
  insert: function(userId, doc) {
    // 只允许登录用户添加帖子
    return !! userId;
  }
});
```

上面的代码使每一个登录了用户都可以发帖子，但也存在以下问题：

1. 登出后的用户仍然可以访问帖子创建页面。
2. 帖子并没有以任何方式与用户进行绑定（没有在服务器上的代码去执行这个）。
3. 允许创建指向相同的 URL 的多个帖子。

下面来解决这些问题

* 帖子创建页面的可访问性

首先阻止已登出的用户看到帖子创建页面，通过定义一个 路由 Hook 。Hook 在路由过程中进行拦截并可能改变路由器的跳转。

我们需要做的是检查用户是否登录，如果他们没有登录，呈现出来的是 accessDenied 模板而不是 postSubmit 模板。 让我们去修改 router.js 文件：

```
var requireLogin = function() {
  if (! Meteor.user()) {
    this.render('accessDenied');
  } else {
    this.next();
  }
}

Router.onBeforeAction(requireLogin, {only: 'postSubmit'});
```

还要创建拒绝访问模板`client/templates/includes/access_denied.html`：

```
<template name="accessDenied">
  <div class="access-denied jumbotron">
    <h2>Access Denied</h2>
    <p>You can't get here! Please log in.</p>
  </div>
</template>
```

上面还有一个问题：

登录，然后尝试刷新页面。你可能注意到，拒绝访问的页面会短暂地出现在帖子创建页面。这是因为在服务器去检测当前用户之前，Meteor 会尽可能快的去渲染模板。

为了避免这个问题（这是一种常见的问题，你将会看到更多去处理客户端和服务器之间错综复杂的延迟），我们将短暂显示一个加载的画面，腾出足够时间让我们去判断用户是否有权访问。

毕竟在这之前，我们不知道用户是否有正确的登录凭证，而我们也不能直接显示 accessDenied 或 postSubmit 模板。

```
var requireLogin = function() {
  if (! Meteor.user()) {
    if (Meteor.loggingIn()) {
      this.render(this.loadingTemplate);
    } else {
      this.render('accessDenied');
    }
  } else {
    this.next();
  }
}
```

* 隐藏链接

当他们注销登录之后就隐藏这个链接，这是最简单的方法去防止用户试图访问不被授权的页面。

```
<ul class="nav navbar-nav">
  {{#if currentUser}}
    <li><a href="{{pathFor 'postSubmit'}}">Submit Post</a></li>
  {{/if}}
</ul>
```

* meteor更好的解决方案？

我们让没有登录的用户无法访问帖子创建页面，并且不允许这样的用户通过使用控制台去创建帖子。然而，仍有一些更多的事情我们需要考虑：

1. 帖子的时间戳。
2. 确保拥有相同 URL 的帖子不能再次创建。
3. 添加帖子的作者的详细信息（ID 、用户名等）。

如果把这些事情放到我们的 submit 事件中去处理。但是实际上，我们很快就会遇到一系列的问题。

1. 对于时间戳，我们并不是总需要依赖用户计算机的时间是否正确。
2. 客户并不知道所有发布到该网站的帖子的 URL 。他们只能知道目前看到的帖子（稍后我们将看看这是如何工作），所以没有办法在每个客户端中确保 URL 是否唯一的。
3. 最后，虽然我们可以在客户端添加用户的详细信息，但是我们不能确保其准确性，因为可能有人会使用浏览器控制台去进行编辑更改。

因为这些问题，最好是保持我们的事件处理方法里面足够简单，如果我们所做的事情超过最基本的插入或更新数据集合，那么可以使用 Meteor 的内置方法 。

Meteor 内置方法是一种服务器端方法提供给客户端调用。其实我们对它并不陌生–事实上在后台， Collection 的 insert、update 和 remove 都属于 Meteor 内置方法。下面看看我们如何自己来创建。

让我们回到 post_submit.js 文件，不再是直接插入到 Posts 集合，我们将调用一个名为 postInsert 的内置方法：

```
Template.postSubmit.events({
  'submit form': function(e) {
    e.preventDefault();

    var post = {
      url: $(e.target).find('[name=url]').val(),
      title: $(e.target).find('[name=title]').val()
    };

    Meteor.call('postInsert', post, function(error, result) {
      // 显示错误信息并退出
      if (error)
        return alert(error.reason);
      Router.go('postPage', {_id: result._id});
    });
  }
});
```

Meteor.call 方法通过第一个参数来调用其方法。你可以为调用方法提供参数（在这种情况下，由表单数据来构建的 post 对象），最后加上一个回调，它将在服务器的方法完成后执行。

Meteor 方法回调总会有两个参数，error 和 result。如果 error 参数由于某种原因存在的话，我们会警告用户（使用 return 来终止回调）。如果正常运行的话，我们将用户转到刚刚创建好的帖子评论页面。

* 安全检查

添加 [audit-argument-checks](http://docs.meteor.com/#full/auditargumentchecks) 包来做安全检查，在`lib/collections/posts.js`中：

```
Posts = new Mongo.Collection('posts');

Meteor.methods({
  postInsert: function(postAttributes) {
    check(Meteor.userId(), String);
    check(postAttributes, {
      title: String,
      url: String
    });
    var user = Meteor.user();
    var post = _.extend(postAttributes, {
      userId: user._id,
      author: user.username,
      submitted: new Date()
    });
    var postId = Posts.insert(post);
    return {
      _id: postId
    };
  }
});
```

因为 Meteor Methods 是在服务器上执行，所以 Meteor 假设它们是可信任的。这样的话，Meteor 方法就会绕过任何 allow/deny 回调。

如果你真的想在服务器端每次 insert、update 或 remove 之前，运行一些代码的话，可以查看 [collection-hooks](https://github.com/matb33/meteor-collection-hooks) 代码包的相关信息。

* 防止重复

唯一性检查：

```
Meteor.methods({
  postInsert: function(postAttributes) {
    check(this.userId, String);
    check(postAttributes, {
      title: String,
      url: String
    });

  // 唯一性检查
    var postWithSameLink = Posts.findOne({url: postAttributes.url});
    if (postWithSameLink) {
      return {
        postExists: true,
        _id: postWithSameLink._id
      }
    }

    var user = Meteor.user();
    var post = _.extend(postAttributes, {
      userId: user._id,
      author: user.username,
      submitted: new Date()
    });

    var postId = Posts.insert(post);

    return {
      _id: postId
    };
  }
});
```

通过在数据库中搜寻是否存在相同的 URL。如果找到，我们 return 返回那帖子的 `_id` 和 `postExists: true` 来让用户知道这个特别的情况。由于我们调用了一个 return，方法就会到此停止，而不会执行 insert 声明，因此优雅地防止了任何重复。

剩下的就是用 postExists 信息通过我们客户端的事件 helper 来显示警告信息：

```
Template.postSubmit.events({
  'submit form': function(e) {
    e.preventDefault();

    var post = {
      url: $(e.target).find('[name=url]').val(),
      title: $(e.target).find('[name=title]').val()
    };

    Meteor.call('postInsert', post, function(error, result) {
      // 向用户显示错误信息并终止
      if (error)
        return alert(error.reason);

      // 显示结果，跳转页面
      if (result.postExists)
        alert('This link has already been posted（该链接已经存在）');

      Router.go('postPage', {_id: result._id});
    });
  }
});
```

* 帖子排序

可以使用 Mongo 数据库的 sort 运算方法，根据这个字段去把对象进行排序，并且标识它们是升序还是降序。

```
Template.postsList.helpers({
  posts: function() {
    return Posts.find({}, {sort: {submitted: -1}});
  }
});
```
