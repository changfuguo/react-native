# react native 代码分离打包

作为一个小前端，九月份接触到RN，说实在的有些小小激动；毕竟可以绕过纷扰的oc和java，做些简单的小demo，其中踩到一些坑，下面说说一个首要的问题-index.ios.bundle的代码分离；

其中困扰很久的一个问题就是代码的分离：rn提供的打包机制将业务代码和RN的lib代码打包到一个文件里，固然没错；仔细看了下，我的业务文件压缩之后的只有370K左右，lib包大小就520K，每次更新代码都白白下载520k？下载速度影响不说，费轱辘呀~~


## 1、代码分离之 react-native-webpack-server

### 1.1 分离打包原理

上网找了下，看到了这个插件[react-native-webpack-server](https://github.com/mjohnston/react-native-webpack-server),这个插件基本上已经实现了代码的分离，

*  其中lib库的代码通过在当前文件夹下生成 `index.ios.js` 和 `index.android.js`里面内容均为，
	`global.React = require('react-native);`然后将该文件的位置传给RN的packager作为入口，里面没有任何业务代码实际上就完成了lib的库的打包，输出还是通过原来的8081端口输出；
* 业务文件的打包；是通过webpack的打包机制，进行打包，端口通过8082端口输出

* 启动自己的打包服务，根据参数去8081拿lib代码，去8082拿webpack打包的业务代码，拼装输出，真正输出的是这个8080端口

### 1.2 分离打包的流程图

上述流程示意图如下，以启动服务为例，如下图所示

![](https://raw.githubusercontent.com/changfuguo/react-native/master/images/how-to-depart-lin-and-app-code-for-rn/rnws.jpg)

### 1.3 如何解决

从上面的流程图看出来，实际上作者在运行期间构造内容只有 `global.React = require('react-native);`的两个空文件，还是让RN本身的packager来生成lib文件，有两个问题要解决

* RN的打包机制打包库文件，如何在webpack打包业务文件忽略RN的库文件呢？
* 打包的lib和文件和库文件如何组织输出给app呢？

***第一个问题***是通过设置webpack的externals实现，extrenal的配置示意如下

```

externales = [{
	"ActivityIndicatorIOS":"commonjs ActivityIndicatorIOS"
}]

```

意思是webpack打包的时候遇到 `require('ActivityIndicatorIOS')`用commjs的规范来获取引用 ，在打包后的文件的脚本如下：

```
function(module, exports) {

	module.exports = require("ActivityIndicatorIOS");

},

```

***第二个问题*** 自己组织了一个server，将lib文件和webpack打包的app文件又+了下，输出给na端


>  其实到这里分离打包的过程基本完成了，剩下的自己可以根据实际内容进行改造了，只是在热更新机制上需要做一些个工作，在`0.8.3`版本里作者没有实现；这个其实无关紧要了，因为 我的目的是要分离lib和app业务文件，目的已经达到了



## 2、改进

### 2.1 存在的问题及解决

* 每次打包都动态生成webpack的externals参数

   这个过程及其漫长，虽然在debug过程中不会再进行变化了；可是退出调试后，再打开，还要在生成externals参数，每次都的等待N秒，索性我就第一次根据RN版本生成externals文件， 直接写成commnjs形式，下次进来直接找让点rn版本对应的externals文件，不存在再重新生成，其目录结构如下
   `./externals/0.18.0/modules.js`
 
 
* 每次的lib代码都继续从8081拉取
   
   其实没必要每次都从8081拉取lib代码，可以在8080的server层做个缓存；因为影响lib代码的因素开发中只有三个platform、dev、minify三个参数，按照这三个参数做缓存就可以了，大大加快debug的速度，
   
   
* 后续持续集成推送lib和app代码到服务端
   
   启示后续如果要将打包的代码进行管理，还有其他一些额外 的工作要做，比如指定当前代码对应的NA端及NA端版本，以及对应的RN版本，否则NA端没有办法更新；为了能扩展利用目前的server服务，讲server的路由部分暴露了出来 
   
 ```
   module.exports = function(ctx){
	var server  = ctx.server;

	context = ctx;

	server.get('/lib.code',assembleLib);

	server.get('/app.code',assembleApp);

	server.get('/all.code',assembleAll);
}
```
可以自己写路由控制，并且以后有推送本地bundle到服务端的话，也可以在这里写路由

### 2.2 react-native-webpackager-server

在对问题做了稍稍的改进之后，搞了一个名字就发布了[react-native-webpackager-server](https://www.npmjs.com/package/react-native-webpackager-server),这里还大力感谢原作者，[mjohnston](https://github.com/mjohnston)

反正我用的还可以，最起码速度加快了，最主要为以后的代码分离做了一个实践；里面有个demo可以仔细看看，如果实在看不懂 哈哈，就看react-native-webpack-server 这个怎么实现的，再来看我稍微改良的这个~~~

