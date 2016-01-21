> 这里记录我在react-native开发过程中遇到的各种问题及相关资源汇总

 
# 1、环境配置

1   官方文档必备：[http://facebook.github.io/react-native/docs/getting-started.html#content](官方文档)


2   maven 国内镜像 及gradle的配置方法
  天朝下载打包依赖实在是慢的要死，改成国内的 刷刷的，再也不用为翻墙发愁了
  [Gradle 修改 Maven 仓库地址](http://www.yrom.net/blog/2015/02/07/change-gradle-maven-repo-url/)
  
----------
  一个简单的办法，修改项目根目录下的build.gradle，将jcenter()或者mavenCentral()替换掉即可：

  allprojects {
      repositories {
          maven{ url 'http://maven.oschina.net/content/groups/public/'}
      }
  }
  
# 2常用组件 

1、顶部是导航栏 下面的当行的内容  

[react-native-scrollable-tab-view](https://github.com/brentvatne/react-native-scrollable-tab-view)


看图

![](https://raw.githubusercontent.com/brentvatne/react-native-scrollable-tab-view/master/demo.gif)
![](https://raw.githubusercontent.com/brentvatne/react-native-scrollable-tab-view/master/demo-fb.gif)

2、Listview向右滑动删除 
[react-native-swipeout](https://github.com/dancormier/react-native-swipeout)

![](https://camo.githubusercontent.com/b48b5dfb4a8ca88cb1408a9a0ff5aecd8f92ff4b/687474703a2f2f692e696d6775722e636f6d2f6f43514c4e46432e676966)

2、字体代替图片

demo期间 为了方便找一个字体做图标，不用放大N多个文件夹下 岂不是更好，于是找了[react-native-icons](https://github.com/corymsmith/react-native-icons)


# 3开源的项目

1 知乎日报 ：[ZhiHuDaily-React-Native](https://github.com/race604/ZhiHuDaily-React-Native)




# 4开发文章

1、[React Native模块桥接详解](http://www.dobest.me/post/react-native-bridge/)

2、[React Native: Android 的打包](http://www.liaohuqiu.net/cn/posts/react-native-android-package/)

3、安装 [React Native for Android 入门老虎](http://www.race604.com/react-native-for-android-start/)

4、 安装包  [React Native for Android 发布独立安装包](http://www.race604.com/rn-android-standalone-apk/)

5、 [React Native for Android 实践 -- 实现知乎日报客户端](http://www.race604.com/react-native-android-practice/)

6、****[整理了一份React-Native学习指南 ](http://www.w3ctech.com/topic/909)*****

7、****[JSPatch – 动态更新iOS APP](http://blog.cnbang.net/works/2767/)

8、[react native的 生命周期](http://www.race604.com/react-native-component-lifecycle/)
 
 本文详细介绍了reat native组件生命周期的几个过程，涉及到props 和state的创建、改变以及何时更改、销毁组件，值得一看
 
9、 [使用 JS 构建跨平台的原生应用：React Native iOS 通信机制初探](http://taobaofed.org/blog/2015/12/30/the-communication-scheme-of-react-native-in-ios/)
   还是有点干货，讲的前端er比较能看懂；打包到机制建议阅读
   
10 、[react-native-webpack-server](https://github.com/mjohnston/react-native-webpack-server)

11、[React Native通信机制详解](http://blog.cnbang.net/tech/2698/)


# 5 相关博主 

1、[Jlog] (http://www.race604.com/)

2、[srain](http://www.liaohuqiu.net/)

3、[browniefed](http://browniefed.com/)

4、 [reactnative](http://www.reactnative.com/)  开源论坛

# 6 开发工具

1、 https://rnplay.org/apps/we3HnA  在线调试react-native

 
![手术室](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/hot.webm)
