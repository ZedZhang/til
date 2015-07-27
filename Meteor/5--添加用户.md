##### 添加用户

使用账号库:

```
meteor add ian:accounts-ui-bootstrap-3
meteor add accounts-password
```

优化layout文件：

```
<template name="layout">
  <div class="container">
    {{> header}}
    <div id="main" class="row-fluid">
      {{> yield}}
    </div>
  </div>
</template>
```

切分出`client/templates/includes/header.html`：

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
        <ul class="nav navbar-nav navbar-right">
          {{> loginButtons}}
        </ul>
      </div>
    </div>
  </nav>
</template>
```

告诉我们的账户系统要求用户需要通过用户名密码来登录，只需配置一个 Accounts.ui 模块到一个名叫 config.js 的文件里面，并把文件放在 client/helpers/ 中：

```
Accounts.ui.config({
  passwordSignupFields: 'USERNAME_ONLY'
});
```

接下来去注册一个帐户：然后“登录”按钮会变成显示你的用户名。

* 原理

在添加 accounts 包以后， Meteor 已经创造了一个新的集合，它可以通过 Meteor.users 去访问。

账户库会“自动发布”了当前登录用户的基本账户信息。如果没有的话，该用户是无法登录到网站的！更重要的是，我们用户的集合似乎在服务器和客户端中并不是包含完全相同的字段。在 Mongo 数据库中，用户拥有大量的数据。比如：

```
> db.users.find()
{
  "_id": "H5kKyxtbkLhmPgtqs",
  "createdAt": ISODate("2015-02-10T08:26:48.196Z"),
  "profile": {},
  "services": {
    "password": {
      "bcrypt": "$2a$10$yGPywo3/53IHsdffdwe766roZviT03YBGltJ0UG"
    },
    "resume": {
      "loginTokens": [{
        "when": ISODate("2015-02-10T08:26:48.203Z"),
        "hashedToken": "npxGH7Rmkuxcv098wzz+qR0/jHl0EAGWr0D9ZpOw="
      }]
    }
  },
  "username": "sacha"
}
```

但是在浏览器中，用户对象的字段就比较少了，可以通过输入这样的命令看到：

```
> Meteor.users.findOne();
Object {_id: "kYdBd9hr3fWPGPcii", username: "tmeasday"}
```

这个例子向我们展示了本地集合是如何成为一个安全的数据库数据子集。登录用户只能看到那些足够去配合客户端工作（例如：登陆）的集合属性。

根据[文档](http://docs.meteor.com/#meteor_users)， Meteor.users 集合也可以选择性地发布更多的字段。

