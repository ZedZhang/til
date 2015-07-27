##### 安装
`$ curl https://install.meteor.com | sh`
##### 新建项目
`$ meteor create your_project`  
这和rails是一样的。
##### run it
`$ cd microscope`  
`$ meteor`  
打开浏览器访问`http://localhost:3000`即可看到：  
"Welcome to Meteor"
##### 添加代码包(lib)
比如加入bootstrap：  
`$ meteor add twbs:bootstrap`

关于lib的类型： 

* Meteor 核心代码本身分成多个核心代码包（core package），每个 Meteor 应用中都包含，你基本上不需要花费精力来维护它们
* 常规 Meteor 代码包称为“isopack”，或同构代码包（isomorphic package，意味着它们既能在客户端也能在服务器端工作）。第一类代码包例如 accounts-ui 或 appcache 由 Meteor 核心团队维护，与 Meteor 捆绑在一起。
* 第三方代码包就是其他用户开发的 isopack 上传到 Meteor 的代码包服务器上。你可以访问 Atmosphere 或 meteor search 命令来浏览这些代码包。
* 本地代码包（local package）是自己开发的代码包，保存在 /packages 文件夹中。
* NPM 代码包（NPM package）是 Node.js 的代码包，虽不能直接用于 Meteor，但可以在上述几种代码包中使用

##### 设置项目结构

1. 删除原来目录下自动生成的html,js,css三个文件
2. 新建4个子文件夹分别是
  1. `/client`
  2. `/server`
  3. `/public`
  4. `/lib`
3. 在`/client`中创建`main.html`和`main.js`
4. 在`/client`中创建`/stylesheets/style.css`

文件相关的规则:  

* 在 `/server` 文件夹中的代码只会在服务器端运行。
* 在 `/client` 文件夹中的代码只会在客户端运行。
* 其它代码则将同时运行于服务器端和客户端上。
* 请将所有的静态文件（字体，图片等）放置在 /public 文件夹中。

Meteor加载文件的顺序：  

* 在 `/lib` 文件夹中的文件将被优先载入。
* 所有以 `main.*` 命名的文件将在其他文件载入后载入。
* 其他文件以文件名的字母顺序载入。 

##### some magic?

 `.meteor` 文件夹。这是 Meteor 存储它内部代码的地方，别轻易修改里面的内容。但可以查看 `.meteor/packages` 和 `.meteor/release` 文件，它们分别列出了你安装的所有智能代码包和你使用的 Meteor 版本。

##### 部署( Meteor Up )

###### 安装

`npm install -g mup`

###### 创建部署目录

使用单独的目录出于两个原因：

第一，这可以很好的避免里面包含任何你 Git 存储库的隐藏文件，尤其如果你是在公共代码库去操作。

第二，通过使用多个单独的目录，我们能够并行地进行多个 Meteor Up 管理和配置。这将会用在实际产品的部署以及分段实例的部署。

```
mkdir ~/project-deploy
cd ~/project-deploy
mup init
```

init之后会创建两个文件: `mup.json`和`settings.json`。

`mup.json` 会保存所有我们部署的相关设置，而 `settings.json` 会保存所有应用的相关设置（OAuth token、Analytics token，等等）。

```
{
  //server authentication info
  "servers": [{
    "host": "hostname",
    "username": "root",
    "password": "password"
    //or pem file (ssh based authentication)
    //"pem": "~/.ssh/id_rsa"
  }],

  //install MongoDB in the server
  "setupMongo": true,

  //location of app (local directory)
  "app": "/path/to/the/app",

  //configure environmental
  "env": {
    "ROOT_URL": "http://supersite.com"
  }
}

```

配置相关说明：

`servers` :服务器身份验证，基于密码和基于私钥（PEM）的身份验证，注意确保是否已经安装了[sshpass](https://gist.github.com/arunoda/7790979)

`setupMongo`: MongoDB 配置，可用本地数据库，则设为`true`；也可使用第三方提供商，比如[Compose](https://www.compose.io/)，则设为`false`，并添加`MONGO_URL`环境变量到`mup.json`中的`env`模块。

`app`: Meteor 应用路径，因为 Meteor Up 的配置作用在不同的目录，我们需要通过 app 属性去把 Meteor Up 指回到应用。

###### 配置服务器

`mup setup`

###### 开始部署

`mup deploy`

###### 显示日志信息

`mup logs -f`
