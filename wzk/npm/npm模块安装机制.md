#### 引言
- 本文介绍 npm 模块安装机制的细节；
- 最近在应用angular-cli搭建的项目中引入ng-zorro时遇见奇葩的bug：
  - 必须在ng new appname初始化第一次执行npm install之前在package.json中手动增加@angular/cdk依赖才不报错；
  - 执行npm install后把ng-zorro源码放到项目中报错缺少@angular/cdk时单独安装npm install @angular/cdk还是报错；
  - 删除整个node_modules包，再执行npm install（此时已非第一次执行）还是报错；
  - 报错猜想：
    - npm 缓存；
    - @angular/cdk下载依赖别的node包不能单独下载
  - 为此查阅文档了解了下npm安装机制；

#### 一、从 npm install 说起
- npm install 命令用来安装模块到node_modules目录。
```
$ npm install <packageName>
```
- 安装之前，npm install会先检查，node_modules目录之中是否已经存在指定模块。如果存在，就不再重新安装了，即使远程仓库已经有了一个新版本，也是如此。
- 如果你希望，一个模块不管是否安装过，npm都要强制重新安装，可以使用-f或--force参数。
```
$ npm install <packageName> --force
```
#### 二、npm update
- 如果想更新已安装模块，就要用到npm update命令。
```
$ npm update <packageName>
```
- 它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。

#### 三、registry查询
- npm 模块仓库提供了一个查询服务，叫做 registry ；
- 查询服务网址是 https://registry.npmjs.org/；
- 这个网址后面跟上模块名，就会得到一个 JSON 对象，里面是该模块所有版本的信息
  - 比如，访问 https://registry.npmjs.org/react，就会看到 react 模块所有版本的信息。
  - 它跟下面命令的效果是一样的：
    ```
    $ npm view react
    
    # npm view 的别名
    $ npm info react
    $ npm show react
    $ npm v react
    ```
- registry 网址的模块名后面，还可以跟上版本号或者标签，用来查询某个具体版本的信息。
- 返回的 JSON 对象里面，有一个dist.tarball属性，是该版本压缩包的网址。
    ```
    dist: {
      shasum: '2a57c2cf8747b483759ad8de0fa47fb0c5cf5c6a',
      tarball: 'http://registry.npmjs.org/react/-/react-0.14.6.tgz' 
    },
    ```
- 到这个网址下载压缩包，在本地解压，就得到了模块的源码。
- npm install和npm update命令，都是通过这种方式安装模块的。

#### 四、缓存目录
- npm install或npm update命令，从 registry 下载压缩包之后，都存放在本地的缓存目录。
- 这个缓存目录，在 Linux 或 Mac 默认是用户主目录下的.npm目录，在 Windows 默认是%AppData%/npm-cache。通过配置命令，可以查看这个目录的具体位置。
    ```
    $ npm config get cache
    $HOME/.npm
    ```
- 你最好浏览一下这个目录。
    ```
    $ ls ~/.npm 
    # 或者
    $ npm cache ls
    ```
- 你会看到里面存放着大量的模块，储存结构是{cache}/{name}/{version}。
    ```
    $ npm cache ls react
    ~/.npm/react/react/0.14.6/
    ~/.npm/react/react/0.14.6/package.tgz
    ~/.npm/react/react/0.14.6/package/
    ~/.npm/react/react/0.14.6/package/package.json
    ```
    - 每个模块的每个版本，都有一个自己的子目录，里面是代码的压缩包package.tgz文件，以及一个描述文件package/package.json。
- 除此之外，还会生成一个{cache}/{hostname}/{path}/.cache.json文件；
  - 比如，从 npm 官方仓库下载react模块的时候，就会生成registry.npmjs.org/react/.cache.json文件。
- 这个文件保存的是，所有版本的信息，以及该模块最近修改的时间和最新一次请求时服务器返回的 ETag 。
    ```
    {
      "time":{
        "modified":"2016-01-06T23:52:45.571Z",
        // ...
      },
      "_etag":"\"7S37I0775YLURCFIO8N85FO0F\""
    }
    ```
- 对于一些不是很关键的操作（比如npm search或npm view），npm会先查看.cache.json里面的模块最近更新时间，跟当前时间的差距，是不是在可接受的范围之内。如果是的，就不再向远程仓库发出请求，而是直接返回.cache.json的数据。
- .npm目录保存着大量文件，清空它的命令如下：
    ```
    $ rm -rf ~/.npm/*
    # 或者
    $ npm cache clean
    ```
#### 五、模块的安装过程
- 总结一下，Node模块的安装过程是这样的：
  - 1、发出npm install命令
  - 2、npm 向 registry 查询模块压缩包的网址
  - 3、下载压缩包，存放在~/.npm目录
  - 4、解压压缩包到当前项目的node_modules目录
- 注意，一个模块安装以后，本地其实保存了两份。一份是~/.npm目录下的压缩包，另一份是node_modules目录下解压后的代码。
- 但是，运行npm install的时候，只会检查node_modules目录，而不会检查~/.npm目录。也就是说，如果一个模块在～/.npm下有压缩包，但是没有安装在node_modules目录中，npm 依然会从远程仓库下载一次新的压缩包。
- 这种行为固然可以保证总是取得最新的代码，但有时并不是我们想要的。最大的问题是，它会极大地影响安装速度。即使某个模块的压缩包就在缓存目录中，也要去远程仓库下载，这怎么可能不慢呢？
- 另外，有些场合没有网络（比如飞机上），但是你想安装的模块，明明就在缓存目录之中，这时也无法安装。

#### 六、--cache-min 参数
- 为了解决上面这些问题，npm提供了一个--cache-min参数，用于从缓存目录安装模块。
- --cache-min参数指定一个时间（单位为分钟），只有超过这个时间的模块，才会从 registry 下载。
    ```
    $ npm install --cache-min 9999999 <package-name>
    //它还有另一种写法
    $ npm install --cache-min Infinity <package-name>
    ```
    - 上面命令指定，只有超过999999分钟的模块，才从registry下载。实际上就是指定，所有模块都从缓存安装，这样就大大加快了下载速度。
- 但是，这并不等于离线模式，这时仍然需要网络连接。因为现在的--cache-min实现有一些问题。
  - （1）如果指定模块不在缓存目录，那么 npm 会连接 registry，下载最新版本。这没有问题，但是如果指定模块在缓存目录之中，npm 也会连接 registry，发出指定模块的 etag ，服务器返回状态码304，表示不需要重新下载压缩包。
  - （2）如果某个模块已经在缓存之中，但是版本低于要求，npm会直接报错，而不是去 registry 下载最新版本。
- npm 团队知道存在这些问题，正在重写 cache。并且，将来会提供一个--offline参数，使得 npm 可以在离线情况下使用。
- 不过，这些改进没有日程表。所以，当前使用--cache-min改进安装速度，是有问题的。

#### 七、离线安装的解决方案
- 社区已经为npm的离线使用，提出了几种解决方案。它们可以大大加快模块安装的速度。
- 解决方案大致分成三类:
  - 第一类，Registry 代理。
  - 第二类，npm install替代。
  - 第三类，node_modules作为缓存目录。

#### 先关链接
- http://www.ruanyifeng.com/blog/2016/01/npm-install.html
- https://www.runoob.com/nodejs/nodejs-npm.html