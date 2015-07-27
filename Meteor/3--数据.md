##### 集合( Collection/Model? )

集合是一个特殊的数据结构，它将你的数据存储到持久的、服务器端的 MongoDB 数据库中， 通过发布（publications）和订阅（subscriptions）机制把数据实时同步上行或者下行到连接着的各个用户的浏览器或者Mongo数据库中，使与每一个连接的用户浏览器进行实时地同步。

首先创建文件`lib/collections/posts.js`: 

```
Posts = new Mongo.Collection('posts');
```

代码所在的目录既不是 `client/` 也不是 `server/` 所以 `Posts` 会共同存在运行在服务器和客户端。 然而，这个集合的使用在两种环境下十分不同： 

* 在服务器，集合有一个任务就是和 Mongo 数据库联络，读取任何数据变化。 在这种情况下，它可以比对标准的数据库。
* 在客户端，集合是一个安全拷贝来自于实时一致的数据子集。客户端的集合总是（通常）透明地实时更新数据子集。

注意这里不需要为变量添加`var`，原因是在 Meteor 中，关键字 `var` 限制对象的作用域在文件范围内，而这里我们想要 `Posts` 作用于整个应用范围内。

* 服务端的集合，可以像 API 一样操作 Mongo 数据库。在服务器端的代码，你可以写像 `Posts.insert()` 或 `Posts.update()` 这样的 Mongo 命令，来对 Mongo 数据库中的 `posts` 集合进行操作。
* 客户端的集合，当你在客户端声明 `Posts = new Mongo.Collection('posts');` 你实际上是创建了一个本地的，在浏览器缓存中的真实的 Mongo 集合。 当我们说客户端集合被"缓存"是指它保存了你数据的一个子集，而且对这些数据提供了十分快速的访问。
* 客户端的那些数据是被存储在**浏览器内存中**的，也就是说访问这些数据几乎不需要时间，不像去服务器访问 Posts.find() 那样需要等待，因为数据事实上已经载入了。

###### 存储数据

网络应用有三种基本方式保存数据，各种方式有不同的角色：

* 浏览器内存：像 JavaScript 变量的这些数据会保存在浏览器内存中，意味着他们不是永久性的：它们存在于当前浏览器标签中，当标签关闭后它们会消失。
* 浏览器存储：浏览器也可存储较为永久性的数据，使用 cookies 或本地存储 Local Storage。虽然数据会在不同 session 间保持，但是只是针对于当前用户（包括标签之间）但不能轻易地共享给其他用户。
* 服务器端数据库：你想永久保存数据并且提供给多个用户的最好方法是数据库（MongoDB 是 Meteor 应用默认的方案）。

Meteor 使用所有三种方式，有时会从一个地方同步数据到另一个地方（我们会马上看到）。话虽如此，数据库仍然是包含数据主副本的“规范化的”数据源。

###### MiniMongo

Meteor 的客户端 Mongo 的技术实现被成为 MiniMongo。它目前还不是一个完美的实现，而且你会发现偶尔 Mongo 的功能在这里不能实现。不过，通过MiniMongo，Meteor 把你的数据拿出一部分子集复制到客户端。这样导致两个结果：

第一， 服务器不再发送 HTML 代码到客户端，而是发送真实的原始数据，让客户端决定如何处理[线传数据](http://docs.meteor.com/#sevenprinciples)。

第二，你可以不必等待服务器传回数据，而是立即访问甚至修改数据（[延迟补偿 latency compensation](http://docs.meteor.com/#sevenprinciples)）。

##### 调试 ( Console )

* 服务器端 `console.log()` 会输出到终端中
* 客户端的 `console.log()` 会输出到浏览器控制台( Browser Console )
* 在终端用 `meteor shell` 调用 Meteor Shell， 使你直接接触到应用的服务器端代码。
* 从终端由 `meteor mongo` 或者 `mrt mongo` 来启动 Mongo Shell，你可以在这里直接操作 App 的数据库.

##### 动态数据

前面我们基本把MVC基本都介绍了一遍，现在需要把M(Collection)和C(模板Helper)串联起来实现实时数据。

首先在数据库中插入一些起始数据，如果你之前操作过数据库，可以先通过`meteor reset`清空数据库，然后`meteor`重启应用。创建`server/fixtures.js`: 

```
if (Posts.find().count() === 0) {
  Posts.insert({
    title: 'Introducing Telescope',
    url: 'http://sachagreif.com/introducing-telescope/'
  });

  Posts.insert({
    title: 'Meteor',
    url: 'http://meteor.com'
  });

  Posts.insert({
    title: 'The Meteor Book',
    url: 'http://themeteorbook.com'
  });
}
```

现在数据已经有了，我们只需要在原来的`postsList.helpers`中，把虚构的数据`postsData`删掉，然后改成: 

```
Template.postsList.helpers({
  posts: function() {
    return Posts.find();
  }
});
```

##### autopublish

事实上，之所以实现了实时数据更新，因为我们默认使用了`autopublish`这个包，它简单地把整个集合分享给所有连接的客户端。这个可不是我们期望的样子，所以让我们去掉它。

`meteor remove autopublish`

现在发现所有的posts都不见了，我们来简单地实现一个autopublish。首先建立一个简单的 `Publish()` 函数，它仅仅返回一个反映所有帖子的游标。新建文件`server/publications.js`：

```
Meteor.publish('posts', function() {
  return Posts.find();
});
```

在客户端我们需要订阅这个发布。我们仅仅需要增加这样一行到 `main.js` 文件中：

`Meteor.subscribe('posts');`

OK，posts又回来了。

-----

题外话，接下来的笔记会简洁一些。

-----

##### 发布与订阅

* 选择性发布（恩，这是我自己造的词，意思是只发布我给你看的内容）：

```
// 在服务器端
Meteor.publish('posts', function(author) {
  return Posts.find({flagged: false, author: author});
});
```

```
// 在客户端
Meteor.subscribe('posts', 'bob-smith');
```

* 查找

```
// 在客户端
Template.posts.helpers({
  posts: function(){
    return Posts.find({author: 'bob-smith', category: 'JavaScript'});
  }
});
```

* 发布部分字段

```
Meteor.publish('allPosts', function(){
  return Posts.find({}, {fields: {
    date: false
  }});
});
```
上面的代码只发布作者是 Tom 的帖子，并且隐藏 date 日期字段。

